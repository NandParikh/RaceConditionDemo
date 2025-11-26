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

