# protected_audience_api_sample

> [!WARNING]
> Protected Audience API は、2025年10月に Google が廃止を発表した（[Update on Plans for Privacy Sandbox Technologies](https://privacysandbox.google.com/blog/update-on-plans-for-privacy-sandbox-technologies)）。Chrome からの削除の状況は [Chrome Platform Status](https://chromestatus.com/feature/6552486106234880) を参照。このリポジトリは保守しておらず、アーカイブしている。

以下の流れを実装したprotected-audience-apiのサンプル
1. InterestGroupへの参加
2. auctionの実行(runAdAuction,generateBid)
3. 広告配信
4. レポーティング(reportWin, reportEvent)

## setup
1. 次のオプションつけてchromeの再起動（attestation checkの無効化）`--disable-features=EnforcePrivacySandboxAttestations`
    * e.g. `open -a /Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --args --disable-features=EnforcePrivacySandboxAttestations` (MacOS)
    * ref : https://github.com/privacysandbox/attestation?tab=readme-ov-file
    * error : `Attestation check for Protected Audience on https://localhost:44303 failed.`
    * 本番では enrollment を行い、送られてくる attestation ファイルを `https://example.com/.well-known/privacy-sandbox-attestations.json` のように対象サイトの `.well-known` に置く必要がある
    * ref : https://github.com/privacysandbox/attestation#the-privacy-sandbox-enrollment-attestation-model
    * ref : https://github.com/privacysandbox/attestation/blob/main/how-to-enroll.md
2. privacy sandbox apiの有効化 `chrome://flags/#privacy-sandbox-ads-apis`
3. `https://localhost:4430*`に対する自己証明書登録
    * `https://localhost:44301/advertiser/index.html` safariでアクセスし以下に従って登録(MacOS)
        * 今回の場合は証明書の名前は `root@buildkitsandbox`
        * ref : https://qiita.com/colomney/items/887f9ea7b68a3b427060
        * error : `Worklet error: Failed to load https://... error = net::ERR_CERT_AUTHORITY_INVALID.`
      * 証明書をinstallした後に信頼する必要があることに注意
        * 再起動などでこれがリセットされる可能性がある.  

## startup
1. `docker-compose up -d --build`
    * re-build with no cache
    * `docker-compose build --no-cache`
    * `docker-compose up -d`
2. `cd kvsapi`
3. `./mvnw clean package`
4. `docker cp target/dsp_kvs_api.war paapi-container:/kvsapi/`
5. `docker exec -it paapi-container bash`
6. `java -jar /kvsapi/dsp_kvs_api.war --server.port=8081`
7. `https://localhost:44300/home/index.html` chromeでrootページにアクセス
8. `https://localhost:44301/advertiser/index.html` interestGroupに参加
9. `https://localhost:44302/publisher/index.html` 広告が表示されれば成功

## hosts
- ROOT : `localhost:44300` -> `localhost:8080`
- Advertiser : `localhost:44301` -> `localhost:8080`
- Publisher : `localhost:44302` -> `localhost:8080`
- DSP : `localhost:44303` -> `localhost:8081`
- SSP : `localhost:44304` -> `localhost:8080`

## 関連
- [WICG/turtledove#1303](https://github.com/WICG/turtledove/pull/1303)：Fenced Frames Ads Reporting の仕様に、allowed reporting origins を含む Interest Group の例を追加
