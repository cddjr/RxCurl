# RxCurl
```
//请求testGet这个api
RxCurl::RxCurlProvider provider;
disposeBag << provider.request(BaiduApi::testGet())
  .subscribe([](const std::string& body) {
    TRACE("BaiduApi::testGet OnNext: %s\n", body.substr(0, 32).c_str());
  }, [](std::exception_ptr ep) {
    try { std::rethrow_exception(ep); }
    catch (const std::exception& ex) {
      TRACE("BaiduApi::testGet OnError: %s\n", ex.what());
    }
  }, [] {
    TRACE("BaiduApi::testGet OnCompleted\n"); }
  );
```
***
```
//400毫秒后在background线程中取消网络请求
DispatchQueue.global(DispatchQoS::background).asyncAfterNow(std::chrono::milliseconds(400), [] {
  disposeBag.reset();
  //然后回到主线程打印log
  DispatchQueue.main.async([] {
    TRACE("disposeBag.reset!\n");
  });
});
```
***
```
//测试API的定义
struct BaiduApi : RxCurl::TargetType {
  BaiduApi() {
    baseURL = "http://www.baidu.com";
    method = RxCurl::Method::get;
  }

  static auto testGet() {
    auto api = BaiduApi();
    api.task = std::make_shared<RxCurl::RquestTask::requestPlain>();
    return api;
  }

  static auto testPost(const std::string& id) {
    auto api = BaiduApi();
    RxCurl::RquestTask::Parameters params = { { "id", id } };
    api.task = std::make_shared<RxCurl::RquestTask::requestParameters>(params);
    api.path = "index.htm";
    api.method = RxCurl::Method::post;
    return api;
  }
};
```
