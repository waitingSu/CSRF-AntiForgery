# CSRF:AntiForgery

### checkmarx弱點掃描
```markdown
起因於程式上架需求必須先進行checkmarx的掃瞄，其中被掃出了CSRF的弱點，其中CSRF(Cross-Site Request Forgery)跨網站要求偽造 ，CSRF利用的是網站對用戶網頁瀏覽器的信任，
是指攻擊者通過一些技術手段欺騙使用者的瀏覽器去訪問一個自己曾經認證過的網站，並在其中添加內容，因為瀏覽器曾經認證過該網站，
受訪的網站便會認為該請求為真正的使用者所發出的，這屬於web身分驗證的漏洞，指的是該身分驗證是由特定瀏覽器所發出但是卻不能保證為是真正使用者本身發出的。
``` 
![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/4d33530f-3c0f-4839-8507-782bd5118e05)

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/1dd8f063-4ac4-4f03-865d-ec0892c6be5a)

```markdown
為了防堵被掃出來的弱點，在這次的處理上就使用了AntiForgery的方法來進行處理，AntiForgery是微軟提出用來防堵CSRF的技術，
在ASP.NET MVC中使用AntiForgery token，或稱request verification tokens，主要流程為:

1.客戶端請求一個包含form的HTML頁面。
2.伺服器在回應中須包含兩個tokens。一個token作為cookie發送。另一個則在隱藏的form字段中。令牌是隨機生成的，因此攻擊者無法猜測其值。
3.當客戶端提交提form時，它必須將兩個token發送回伺服器端。客戶端將cookie token作為cookie發送，並在form中發送form token。（當用戶提交表單時，瀏覽器客戶端會自動執行此操作。）
4.如果請求不包含這兩個tokens，伺服器端將不允許該請求。
```


