# CSRF:AntiForgery#

### checkmarx弱點掃描###
    
是指攻擊者通過一些技術手段欺騙使用者的瀏覽器去訪問一個自己曾經認證過的網站，並在其中添加內容，因為瀏覽器曾經認證過該網站，
受訪的網站便會認為該請求為真正的使用者所發出的，這屬於web身分驗證的漏洞，指的是該身分驗證是由特定瀏覽器所發出但是卻不能保證為是真正使用者本身發出的。
  
![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/4d33530f-3c0f-4839-8507-782bd5118e05)

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/1dd8f063-4ac4-4f03-865d-ec0892c6be5a)

      為了防堵被掃出來的弱點，在這次的處理上就使用了AntiForgery的方法來進行處理，AntiForgery是微軟提出用來防堵CSRF的技術，
透過
  

