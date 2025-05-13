# marvtxt


class XmlHttpRequestWrapperForCustom {
    constructor() {
        this._xhr = new XMLHttpRequest();
        this._isRetry = false;

        // 응답 처리
        this._xhr.addEventListener('load', async () => {
            if (this._xhr.status === 401 && !this._isRetry) {
                console.warn('[Wrapper] 401 received, attempting token refresh...');
                this._isRetry = true;

                try {
                    await this.refreshToken(); // 비동기 토큰 갱신
                    const newToken = await this.getToken();
                    //this._xhr = new XMLHttpRequest(); // 새로 객체 생성
                    //this._isRetry = false;
                    await this.retryRequest()

                    console.warn('[Wrapper] Ready for retry, please resend manually.');
                    // 또는 자동으로 open/send 정보를 저장하고 재호출해도 됨
                } catch (err) {
                    console.error('[Wrapper] Token refresh failed:', err);
                }
            }
        });
    }

    getXhr() {
        return this._xhr;
    }

    open(method, url, async = true, user = null, password = null) {
        console.log('[CustomXHR] open:', method, url);

        this._method = method;
        this._url = url;
        return this._xhr.open(method, url, async, user, password);
    }

    send(body = null) {
        console.log('[CustomXHR] send:', body);
        this._body = body;
        return this._xhr.send(body);
    }

    setRequestHeader(header, value) {
        return this._xhr.setRequestHeader(header, value);
    }

    async retryRequest() {
        if (!this._method || !this._url) {
            console.error('[Wrapper] Missing previous request data.');
            return;
        }

        this._xhr = new XMLHttpRequest();
        await this.prepare(); // 토큰을 갱신 후 설정

        this.open(this._method, this._url);
        this.send(this._body ?? null);
    }

    async refreshToken() {
        //try {
        //    const response = await fetch('/refresh-token', { method: 'POST' });
        //    if (!response.ok) {
        //        throw new Error('Token refresh failed');
        //    }
        //    return response.json(); // 갱신된 토큰 반환
        //} catch (err) {
        //    console.error('[Wrapper] Token refresh failed:', err);
        //}

        return "refreshAccessToken";
    }

    async getToken() {
        //try {
        //    const response = await fetch('/get-token', { method: 'GET' });
        //    if (!response.ok) {
        //        throw new Error('Failed to fetch token');
        //    }
        //    return response.json(); // 토큰 반환
        //} catch (err) {
        //    console.error('[Wrapper] Failed to get token:', err);
        //}

        return "11";
    }

    async prepare() {
        const token = await this.getToken();
        this._xhr.setRequestHeader('Authorization', "Bearer " + token);
    }
}




---------------------------------------------------------------------------------------

test.ashx

using System.IO;

public void ProcessRequest(HttpContext context)
{
    context.Response.ContentType = "application/json";

    if (context.Request.HttpMethod == "POST")
    {
        using (StreamReader reader = new StreamReader(context.Request.InputStream))
        {
            string requestBody = reader.ReadToEnd();
            context.Response.Write("{ \"received\": " + requestBody + " }");
        }
    }
    else
    {
        string name = context.Request.QueryString["name"] ?? "Guest";
        context.Response.Write("{ \"message\": \"Hello, " + name + "!\" }");
    }
}

---------------------------------------------------------------------------------------


fetch("/MyHandler.ashx?name=Taehyeon", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ name: "Taehyeon" })
})
.then(response => response.json())
.then(data => console.log("Response:", data))
.catch(error => console.error("Error:", error));

---------------------------------------------------------------------------------------------

위에는 안되고 밑에는 되고 차이가뭔지
try {
    const response = fetch('/CoviWeb/approval/rms/services/GetAccessToken.ashx', { method: 'POST' });
    if (!response.ok) {
        throw new Error('Token refresh failed');
    }
    return response.json(); // 갱신된 토큰 반환
} catch (err) {
    console.error('[Wrapper] Token refresh failed:', err);
}


//$.ajax({
//    url: "/CoviWeb/approval/rms/services/GetAccessToken.ashx",
//    type: "POST",
//    dataType: "json",
//    data: null,
//    async: false, //동기 방식으로 진행함
//    error: function (error) {
//        alert('데이터 조회에 실패하였습니다. 다시 시도해 주세요.');
//    }
//})
//    .done(function (response) {
//        debugger;
//        console.log(response);
//        alert(11);
//    });
