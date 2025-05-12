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
