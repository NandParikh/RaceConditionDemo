---
What is Race condition?
---
- Race conditions happen when multiple threads access shared data at the same time, leading to unpredictable results.

```Swift
import UIKit
/*
 Output:
 Without serial queue or lock
    ❗ Final Balance: 18720
    Expected should be: 20000
 
 With serial queue or lock
    ❗ Final Balance: 2000
    Expected should be: 20000
 
 Alternate solutions:
    let semaphore = DispatchSemaphore(value: 1) // priority
    semaphore.wait()
    do logic
    semaphore.signal()
 */

class Bank {
    var balance = 0
    //private let lock = NSLock()

    private let queue = DispatchQueue(label: "bank.queue")
     func deposit(_ amount: Int) {
         
         //lock.lock()
         queue.sync {
             let temp = balance
             balance = temp + amount
         }
        //lock.unlock()
     }
 }

class ViewController: UIViewController {
    
    @IBAction func btnClick(_ sender: UIButton) {
        
        let bank = Bank()
        
        DispatchQueue.global().async {
            for _ in 1...10000 { bank.deposit(1) }
        }
        
        DispatchQueue.global().async {
            for _ in 1...10000 { bank.deposit(1) }
        }
        
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {
            print("❗ Final Balance:", bank.balance)
            print("Expected should be: 20000")
        }
    }
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
    }
}

```

| Method               | Good                        | Avoid when                      |
| -------------------- | --------------------------- | ------------------------------- |
| NSLock               | Simple, fast                | Can deadlock if misused         |
| Semaphore            | General concurrency control | More complex for simple locking |
| Serial DispatchQueue | Clean, easy                 | Slight overhead                 |
| Actor                | Modern & safest             | Requires async/await            |


You’re right — this code does create a race condition if the queue.sync is removed or if access to balance is not isolated — because multiple threads can read & write simultaneously.

But in modern Swift, the cleanest & safest solution is using an actor — because actors automatically provide data isolation without needing locks or DispatchQueues.

⸻

✅ Actor-based Version (Best modern Swift)
```Swift
actor Bank {
    var balance = 0
    
    func deposit(_ amount: Int) {
        balance += amount
    }
}
```
And the caller:
```Swift
class ViewController: UIViewController {

    @IBAction func btnClick(_ sender: UIButton) {

        let bank = Bank()

        Task.detached {
            for _ in 1...10000 { await bank.deposit(1) }
        }

        Task.detached {
            for _ in 1...10000 { await bank.deposit(1) }
        }

        Task {
            try await Task.sleep(nanoseconds: 1_000_000_000)
            let finalBalance = await bank.balance
            print("❗ Final Balance:", finalBalance)
            print("Expected should be: 20000")
        }
    }
}

```
⸻

Why actor solves the race condition

Inside an actor:
	- Only one task at a time can access its stored properties.
	- No two deposit() calls run simultaneously.
	- You don’t need locks
	- You don’t need queues
	- You don’t need sync / barriers

This is the guarantee:

Actor ensures serialized access to its state.

⸻

Explanation for Interview 🔥

If asked: Why use actor instead of DispatchQueue or locks?

Answer:

Actors provide structured concurrency and automatic data isolation. Unlike locks or queues, actors eliminate human error — you cannot accidentally access balance concurrently. The compiler enforces thread-safe access using await. This results in safer concurrent code with simplified syntax.

⸻

Accessing Actor Properties

❗ Direct property read also requires await
```Swift
let value = await bank.balance
```
Because reading is also part of state access.

⸻

Important: Calling actor from UI thread

You must use async:
```Swift
Task {
    let b = await bank.balance
    print(b)
}
```

⸻

Optional — Make deposit return the updated balance
```Swift
actor Bank {
    private(set) var balance = 0
    
    @discardableResult
    func deposit(_ amount: Int) -> Int {
        balance += amount
        return balance
    }
}
```
Usage:
```Swift
let value = await bank.deposit(10)
```

⸻

Summary

Approach	Safe	Easy	Compiler-checked	Recommended in 2025
Using locks (NSLock)	Yes	❌	❌	❌
Using GCD queue	Yes	✔️	❌	❌
Using actor	✔️	✔️✔️✔️	✔️	⭐ BEST ⭐

---
