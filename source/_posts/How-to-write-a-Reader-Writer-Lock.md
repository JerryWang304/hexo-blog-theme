---
title: How to write a Reader-Writer Lock
date: 2017-04-04 08:01:48
tags:
- Rust
- Operating System
---
This is homework of my operating system course. Understanding the basic principle of Reader-Writer lock is not difficult. It is Rust that causes me to spend days working on it.  

### Reader-Writer Lock
RWLock allows multiple readers threads to simultaneously access the data. But only one writer is allowed to do so at any one time. As usual, to writer such a lock, we need a lock (maybe more) and conditional variables.

What are so special of this lock that we need to consider
carefully ? What are the variables and data structures we may use to implement it ?

- Firstly, we need to consider when the readers/writers should wait. For a reader, it is definitely not allowed to read the data if there is some active writer modifying the shared data and vice versa.
- Secondly, we have to think about how to wake up other threads when the current reader/writer is done. In this homework we are required to implement reader-prefered/writer-prefered and FIFO/LIFO. By the way, the rwlock in std is not so complicated.
- Finally, to those who are not familiar with Rust yet, it will be suffering to write it.

### Using Rust
The structure of RWLock is,
```Rust
pub struct RwLock<T> {
    mutex: Mutex<()>,
    data: UnsafeCell<T>,
    // Hao Chen's requirements
    pref: Preference,
    order: Order,

    state_vars: UnsafeCell<Vars>,
}

struct Vars {

    // state variables
    waiting_readers: Vec<Rc<Box<Condvar>>>,
    waiting_writers: Vec<Rc<Box<Condvar>>>,
    active_readers: u32,
    active_writers: u32,
}
```
**mutex** provides the exclusive access of RwLock. We need this because we don't want to be disturbed when modify something.

**data** is what the reader and writer are looking for. We use UnsafeCell because we need interior mutability. Read [this](https://doc.rust-lang.org/std/cell/struct.UnsafeCell.html) if you want to learn more.

** state_vars ** means state variables that including the waiting_writers/readers and active_writers/reader currently. In order to implement FIFO/LIFO, we give each thread a conditional variable that is used to wake the thread up in the future according to FIFO/LIFO. For instance, if I want to wake the first write up, which is waiting because readers are reading, we just call `waiting_writers[0].notify_one()`. Then the thread blocked on this conditional variable will continue and decide to move on or still keep on waiting.

The two enums,
```Rust
pub enum Preference {
    /// Readers-preferred
    /// * Readers must wait when a writer is active.
    /// * Writers must wait when a reader is active or waiting, or a writer is active.
    Reader,
    /// Writers-preferred:
    /// * Readers must wait when a writer is active or waiting.
    /// * Writer must wait when a reader or writer is active.
    Writer,
}

/// In which order to schedule threads
pub enum Order {
    /// First in first out
    Fifo,
    /// Last in first out
    Lifo,
}
```
It is easy to write `write_should_wait()` and `read_should_wait()` based on the annotation in Preference. I won't bother to elaborate on this.

The reader,
```Rust
pub fn read(&self) -> Result<RwLockReadGuard<T>, ()> {
    let mut guard = self.mutex.lock().unwrap();
    let cv = Rc::new(Box::new(Condvar::new())); // point to a condvar
    unsafe {
        (*self.state_vars.get()).waiting_readers.push(cv.clone());
    }

    while self.read_should_wait() {
        guard = cv.wait(guard).unwrap();
    }

    unsafe {

        //(*self.state_vars.get()).waiting_readers.pop();
        match self.order {
            Order::Fifo => {
                (*self.state_vars.get()).waiting_readers.remove(0);
            },
            Order::Lifo => {
                (*self.state_vars.get()).waiting_readers.pop();
            }
        }

        (*self.state_vars.get()).active_readers += 1;

    }

    Ok(
        RwLockReadGuard {
            lock: &self
        }
    )

}
```
After we gain the lock, we have the exclusive access to the RWLock. Firstly, we need a new conditional variable as explained before. We put it on the heap to make sure it is alive. And we put the pointer into a Rc because we have to push it into a vector. If we don't use a Rc, after push cv into the vector, we can't access cv anymore. Don't forget to pop cv out!

We implement the Drop trait for RWLockReadGuard/RWLockWriteGuard, which is just a reference of the RWLock.
```Rust
/// Releases the read lock
impl<'a, T> Drop for RwLockReadGuard<'a, T> {

    fn drop(&mut self) {
        let guard = self.lock.mutex.lock().unwrap();
        unsafe {
            if (*self.lock.state_vars.get()).active_readers > 0 {
                (*self.lock.state_vars.get()).active_readers -= 1;
            }
            self.lock.wakeup_other_threads();
        }

    }

}
```
The main part is wakeup function. It is not so hard.
```Rust
pub fn wakeup_other_threads(&self) {
        //unsafe {
        //    println!("{:?}", (*self.state_vars.get()).waiting_writers);
        //}
        match self.pref {
            Preference::Reader => {
                // firstly weak up readers
                unsafe {
                    let ref mut waiting_readers =  (*self.state_vars.get()).waiting_readers;// vector
                    let ref mut waiting_writers =  (*self.state_vars.get()).waiting_writers;
                    match self.order {
                        Order::Lifo => {

                            if waiting_readers.len() > 0 {

                                let len = waiting_readers.len();
                                for i in 0..len {
                                    waiting_readers[len-1-i].notify_one();
                                }

                            }else if waiting_writers.len() > 0 {

                                waiting_writers[waiting_writers.len()-1].notify_one();

                            }

                        },
                        Order::Fifo => {
                            if waiting_readers.len() > 0 {

                                let len = waiting_readers.len();
                                for i in 0..len {
                                    waiting_readers[i].notify_one();
                                }


                            }else if waiting_writers.len() > 0 {
                                waiting_writers[0].notify_one();

                            }
                        },
                    }
                }            
            },
            Preference::Writer => {
                // firstly weak up writers
                unsafe {
                    let ref mut waiting_writers = (*self.state_vars.get()).waiting_writers;// vector
                    let ref mut waiting_readers = (*self.state_vars.get()).waiting_readers;// vector

                    match self.order {
                        Order::Lifo => {

                            if waiting_writers.len() > 0 {
                                waiting_writers[waiting_writers.len()-1].notify_one();

                            }else if waiting_readers.len() > 0 {
                                let len = waiting_readers.len();
                                for i in 0..len {
                                    waiting_readers[len-1-i].notify_one();
                                }                     

                            }
                        },
                        Order::Fifo => {
                            if waiting_writers.len() > 0 {
                                //println!("&&& {} writers are waiting", waiting_writers.len());
                                waiting_writers[0].notify_one();
                                //println!("&&&& wake one writer &&&&");

                            }else if waiting_readers.len() > 0 {
                                let len = waiting_readers.len();
                                for i in 0..len {
                                    waiting_readers[i].notify_one();
                                }  

                            }                            
                        },
                    }                           
                }
            }
        }
    }
```

See the source code [here](https://github.com/JerryWang304/RWLock)