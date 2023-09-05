# CSRF:AntiForgery

## checkmarx弱點掃描

　　起因於程式上架需求必須先進行checkmarx的掃瞄，其中被掃出了CSRF的弱點，其中CSRF(Cross-Site Request Forgery)跨網站要求偽造 ，<br>
CSRF利用的是網站對用戶網頁瀏覽器的信任，是指攻擊者通過一些技術手段欺騙使用者的瀏覽器去訪問一個自己曾經認證過的網站，<br>
並在其中添加內容，因為瀏覽器曾經認證過該網站，受訪的網站便會認為該請求為真正的使用者所發出的，這屬於web身分驗證的漏洞，指的是該身分驗證是由特定瀏覽器所發出但是卻不能保證為是真正使用者本身發出的。<br>

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/4d33530f-3c0f-4839-8507-782bd5118e05)

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/1dd8f063-4ac4-4f03-865d-ec0892c6be5a)


　　為了防堵被掃出來的弱點，在這次的處理上就使用了AntiForgery的方法來進行處理，AntiForgery是微軟提出用來防堵CSRF的技術，<br>
在ASP.NET MVC中使用AntiForgery token，或稱request verification tokens，主要流程為:<br>

1.客戶端請求一個包含form的HTML頁面。<br>

2.伺服器在回應中須包含兩個tokens。一個token作為cookie發送。另一個則在隱藏的form字段中。令牌是隨機生成的，因此攻擊者無法猜測其值。<br>

3.當客戶端提交提form時，它必須將兩個token發送回伺服器端。客戶端將cookie token作為cookie發送，並在form中發送form token。（當用戶提交表單時，瀏覽器客戶端會自動執行此操作。）<br>

4.如果請求不包含這兩個tokens，伺服器端將不允許該請求。<br>

　　在HTML頁面的處理方法主要可以分為兩種，第一種是透過MVC Form發出表單請求的處理，另一個則是ajax請求的驗證。

### MVC Form的AntiForgery token

　　第一種Form的驗證在MVC本身就有內建AntiforgeryToken()的方法，只需要在Form之中呼叫@HTML.AntiforgeryToken()便可夾帶，
並且在對應的contorller中加入[ValidateAntiForgeryToken]的Filter即可。

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/e26df9ba-eff2-4085-bfca-65d2c804c558)

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/938575a3-f70e-4fc1-a6f8-f6c8b0e23e64)

　　實際輸出的token如下，form的元素中會出現一個name為__RequestVerificationToken的隱藏<input>，會隨著form交提時一起夾帶。<br>
	cookie方面的token只要在對應的HTML頁面加入@HTML.AntiforgeryToken()便會自動產生一個名為__RequestVerificationToken的cookie。<br>
	
![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/fe485cc5-0744-4214-849a-117b5e5a11cb)

 ![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/949e5028-5c0e-459b-b8e3-572311a105b2)


 ### Ajax的AntiForgery token

 　　第二種Ajax的AntiForgery token微軟本身並沒有提供內建的方法，是需要自訂義function來產生token，以及另外自訂一個用於驗證的function。<br>
具體作法上會是分別製作成用於產生token的公用元件以及用於驗證的action filter方便進行套用。

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/333c4c01-dd10-4f79-8b97-b7869b7a94ee)

```markedown
public static string GetAntiForgery()
{
	string cookieToken, formToken;
	AntiForgery.GetTokens(null, out cookieToken, out formToken);
	return string.Concat(cookieToken, ":", formToken);
}
```
![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/e4d9f740-cce3-4d7f-8216-eefea02ea622)

```markdwon
public class AjaxValidateAntiForgeryTokenAttribute : FilterAttribute, IAuthorizationFilter
{
	public void OnAuthorization(AuthorizationContext filterContext)
	{
		try
		{
			//只有Ajax 才處理
			if (filterContext.HttpContext.Request.IsAjaxRequest())
			{
				ValidateRequestHeader(filterContext.HttpContext.Request);
			}
			else
			{
				filterContext.HttpContext.Response.StatusCode = 404;
				filterContext.Result = new HttpNotFoundResult();
			}
		}
		catch (HttpAntiForgeryException e)
		{
			throw new HttpAntiForgeryException("Anti forgery token cookie not found");
		}
	}
	
	/// <summary>
	/// 解析前端丟過來的Token 是否正確
	/// </summary>
	/// <param name="request"></param>
	private void ValidateRequestHeader(HttpRequestBase request)
	{
		String cookieToken = String.Empty;
		String formToken = String.Empty;
		String TokenValue = request.Headers["RequestVerificationToken"];
	
		if (!String.IsNullOrWhiteSpace(TokenValue))
		{
			String[] Tokens = TokenValue.Split(':');
			if (Tokens.Length == 2)
			{
				cookieToken = Tokens[0];
				formToken = Tokens[1];
			}
			AntiForgery.Validate(cookieToken, formToken);
		}
	}
}
```
　　GetAntiForgery()是透過AntiForgery.GetTokens直接產生cookie和form的tokens，並且串接起來直接讓ajxa夾帶，<br>
所以接下來只要放在要驗證的ajax頁面中，並在header中夾帶進"RequestVerificationToken"發送出去即可，對應的呼叫的action也需要套用[AjaxValidateAntiForgeryToken]的filter進行驗證。

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/e4056f53-0d6c-4572-a3c1-73b253c08ad0)

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/9442e9c7-66f5-43bd-8b93-bef81807a0fd)

![image](https://github.com/waitingSu/CSRF-AntiForgery/assets/67044426/861c7e09-537d-47ae-80dd-3ab326789638)



 















