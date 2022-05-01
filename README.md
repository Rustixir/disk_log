
# disk_log


disk_log is a **High Throughout, NonBlocking** Disk-based logger


# Way need to disk_log ?

* **Non Blocking** - optimized for tokio and
  **don't block your scheduler** for IO operation 


* **Paging** - disk_log distribute logs records between pages/files
  by `total_page_size`


* **Iterator** - provide asynchronous function `get_page` then 
  can iterate whole page **(its safe for concurrent reading when disk_log write to it)**  


* **High Concurrency** disk_log don't need to
  Mutex/RwLock for sharing between threads, its complete safe


* **Technically** - disk_log inspired by **erlang disk_log**
  that is a heavily used in erlang , by **mnesia database**
  and **Logger** (Logger is production used Logger for Elixir) 




# Example 

```rust

#[tokio::main]
async fn main() {

  let path = ".";
  let name = "service";
  let total_page_size = 1000;

  // Run DiskLog and get session
  let sess = DiskLog::open(path, name, total_page_size)
                             .unwrap()
                             .run_service();
                             


  // Log to page (don't block your scheduler)

  let _ = sess.log(b"Serialized data ......".to_vec()).await;

  let _ = sess.log(b"Serialized data ......".to_vec()).await;



  // very lightweight, just increment counter (mpsc::sender::clone)
  let sess1 = sess.clone();
  
  
  // Spawn task 
  tokio::spawn(async move {              
   
    // log by another task
    let _ = sess1.log(b"Serialized data ......".to_vec()).await;
  });



  // ---------------------------------------------------------------------
  // asynchronous get page
  let page = sess.get_page(1).await;
  
  match page {
       Err(e) => println!("==> {:?}", e),
       // if exist page
       Ok(mut lf) => {
           lf.iter(..)
             .unwrap()
             .for_each(|_record| {
               // consumer record
             })
       }    
   }

}

```



# Author

- DanyalMh 



## License

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at https://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.
