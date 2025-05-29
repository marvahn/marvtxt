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


-------------------------------------------------------------------------------
SELECT SUM(WIP_TOTAL) AS WIP_TOTAL
FROM(
		SELECT	
			SUM(A.HSL * 100) AS WIP_TOTAL
		FROM
			SWIFT..ACDOCA A  WITH(NOLOCK)
			INNER JOIN TB_RPT_COM_ENGAGEMENT E WITH(NOLOCK) ON LEFT(A.PS_POSID,14) = E.ENG_ID AND ENG_LEVEL = '2' AND E.WBS_TYPE IN ('CE','CL','CD','DE')
		WHERE	
			RLDNR = '0L' 
			AND (BUDAT BETWEEN SUBSTRING(@FISCAL_YEAR,1,CHARINDEX('|',@FISCAL_YEAR)-1) 
				AND RIGHT(@FISCAL_YEAR,LEN(@FISCAL_YEAR)-CHARINDEX('|',@FISCAL_YEAR)))
			AND BUDAT <> '00000000'
			AND RACCT IN ('0012200010','0012200020','0012200035','0012200040','0012200060','0012200090')
			AND ((LEFT(LEFT(A.PS_POSID,14),11) <> '' AND LEN(LEFT(LEFT(A.PS_POSID,14),11)) > 7))
			AND (E.PART_PTR_NO = @SABUN)
			--AND (@SABUN = '' OR E.PART_PTR_NO = @SABUN)
		GROUP BY LEFT(PS_POSID,14)--E.PART_PTR_NO

		UNION ALL

		SELECT	
			SUM(A.HSL * 100) AS WIP_TOTAL
		FROM
			SWIFT..ACDOCA A  WITH(NOLOCK)
			INNER JOIN [SWIFT].dbo.[Z_OLD_SAP_ENG] B ON LEFT(LEFT(A.PS_POSID,14),11) = B.ENG_ID
		WHERE	
			RLDNR = '0L' 
			AND (BUDAT BETWEEN SUBSTRING(@FISCAL_YEAR,1,CHARINDEX('|',@FISCAL_YEAR)-1) 
				AND RIGHT(@FISCAL_YEAR,LEN(@FISCAL_YEAR)-CHARINDEX('|',@FISCAL_YEAR)))
			AND BUDAT <> '00000000'
			AND RACCT IN ('0012200010','0012200020','0012200035','0012200040','0012200060','0012200090')
			AND ((LEFT(LEFT(A.PS_POSID,14),11) <> '' AND LEN(LEFT(LEFT(A.PS_POSID,14),11)) = 7))
			AND (B.PART_PTR_NO = @SABUN)
			--AND (@SABUN = '' OR B.PART_PTR_NO = @SABUN)
		GROUP BY LEFT(PS_POSID,14)--B.PART_PTR_NO
) AS TOT
