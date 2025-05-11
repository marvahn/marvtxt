# marvtxt



public class AuthenticationModule : IHttpModule
{
    public void Init(HttpApplication context)
    {
        context.BeginRequest += (sender, e) =>
        {
            var request = context.Request;
            var token = GetTokenFromLocalStorage();
            if (!string.IsNullOrEmpty(token))
            {
                request.Headers["Authorization"] = "Bearer " + token;
            }
        };
    }

    private string GetTokenFromLocalStorage()
    {
        // 로컬 스토리지에서 토큰을 가져오는 로직을 구현 (예: JavaScript를 통한 저장)
        return ""; // 예제에서는 빈 문자열
    }

    public void Dispose() { }
}

-----------------------------

protected void Application_BeginRequest(object sender, EventArgs e)
{
    HttpContext context = HttpContext.Current;
    string token = GetTokenFromLocalStorage(); // 로컬 스토리지에서 토큰 가져오기
    if (!string.IsNullOrEmpty(token))
    {
        context.Request.Headers["Authorization"] = "Bearer " + token;
    }
}

private string GetTokenFromLocalStorage()
{
    // 토큰 저장 방식에 따라 구현 필요
    return ""; // 예제에서는 빈 값 반환
}
