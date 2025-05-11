# marvtxt

public class AuthenticationModule : IHttpModule
{
    public void Init(HttpApplication context)
    {
        context.BeginRequest += async (sender, e) =>
        {
            var request = context.Request;
            string token = GetAccessToken();
            if (!string.IsNullOrEmpty(token))
            {
                request.Headers["Authorization"] = "Bearer " + token;
            }
        };

        context.EndRequest += async (sender, e) =>
        {
            var response = context.Response;
            if (response.StatusCode == 401) // 인증 실패(토큰 만료)
            {
                bool tokenRefreshed = await TryRefreshTokenAsync();
                if (tokenRefreshed)
                {
                    context.Request.Headers["Authorization"] = "Bearer " + GetAccessToken();
                }
            }
        };
    }

    private string GetAccessToken()
    {
        return HttpContext.Current.Session["accessToken"] as string ?? string.Empty;
    }

    private string GetRefreshToken()
    {
        return HttpContext.Current.Session["refreshToken"] as string ?? string.Empty;
    }

    private async Task<bool> TryRefreshTokenAsync()
    {
        string refreshToken = GetRefreshToken();
        if (string.IsNullOrEmpty(refreshToken))
        {
            return false;
        }

        using (HttpClient client = new HttpClient())
        {
            var request = new HttpRequestMessage(HttpMethod.Post, "https://your-api.com/token/refresh")
            {
                Content = new StringContent($"{{\"refreshToken\": \"{refreshToken}\"}}", Encoding.UTF8, "application/json")
            };
            var response = await client.SendAsync(request);
            if (response.IsSuccessStatusCode)
            {
                var jsonResponse = await response.Content.ReadAsStringAsync();
                var tokenData = JsonConvert.DeserializeObject<TokenResponse>(jsonResponse);
                HttpContext.Current.Session["accessToken"] = tokenData.AccessToken;
                HttpContext.Current.Session["refreshToken"] = tokenData.RefreshToken;
                return true;
            }
        }
        return false;
    }

    public void Dispose() { }
}

public class TokenResponse
{
    public string AccessToken { get; set; }
    public string RefreshToken { get; set; }
}
