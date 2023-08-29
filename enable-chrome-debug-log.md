# 开启及查看Chrome日志

## 开启chrome日志

在chrome启动时加参数：

```
chrome --enable-logging
```

这样日志将保存在本地目录中，具体位置为：

```
C:\Users\YOUR_NAME\AppData\Local\Google\Chrome\User Data\chrome_debug.log
```

每次重新启动chrome，此日志将清空。

如果要同时输出到控制台：

```
chrome --enable-logging=stderr
```

## 日志级别及日志过滤

使用`--v`参数控制日志级别：

```
chrome --enable-logging --v=1
```

- v=1: 默认值，输出：INFO, WARNING, ERROR, and VERBOSE0
- v=n: INFO, WARNING, ERROR, and VERBOSE0 VERBOSE1 ... VERBOSEn

verbose日志可能非常多，影响分析日志，可以使用`--vmodule`参数进行过滤，支持通配符，多个条件使用逗号分隔：

```
chrome --enable-logging --vmodule=compositor*=0,layer_tree*=0,display*=0,*=4
```

## 日志内容示例

```log
[5124:22464:0825/181021.933:WARNING:chrome_main_delegate.cc(608)] This is Chrome version 110.0.5449.0 (not a warning)
[5124:22464:0825/181022.039:VERBOSE1:chrome_browser_cloud_management_controller.cc(94)] DM token = none
[5124:22464:0825/181022.039:VERBOSE1:chrome_browser_cloud_management_controller.cc(100)] Enrollment token = 
[5124:22464:0825/181022.039:VERBOSE1:chrome_browser_cloud_management_controller.cc(101)] Client ID = ce6a36a6-f4c1-4bf1-aaca-c3d55ad1a6f8
[5124:22464:0825/181022.039:VERBOSE2:policy_service_impl.cc(508)] Policy refresh complete
[5124:22464:0825/181022.077:VERBOSE1:webrtc_event_log_manager.cc(96)] WebRTC remote-bound event logging enabled.
[5124:22464:0825/181022.081:VERBOSE1:pref_proxy_config_tracker_impl.cc(187)] 000013C4000D8900: set chrome proxy config service to 000013C4000C01C0
[5124:17608:0825/181022.093:VERBOSE1:media_stream_manager.cc(1139)] MSM::InitializeMaybeAsync([this=000013C400039A00])
[5124:17608:0825/181022.093:VERBOSE1:media_stream_manager.cc(1139)] MDM::MediaDevicesManager()
[5124:17608:0825/181022.093:VERBOSE1:media_stream_manager.cc(1139)] MSM::MediaStreamManager([this=000013C400039A00]))
[5124:22464:0825/181022.095:VERBOSE1:install_util.cc(217)] No existing Chrome install found.
[5124:22464:0825/181022.096:VERBOSE3:browser_switcher_policy_migrator.cc(51)] Migrating Legacy Browser Support extension policies...
[5124:22464:0825/181022.096:VERBOSE3:browser_switcher_policy_migrator.cc(61)] BrowserSwitcherEnabled is false, aborting policy migration.
[5124:22464:0825/181022.096:VERBOSE2:policy_service_impl.cc(508)] Policy refresh complete
[5124:22464:0825/181022.102:VERBOSE2:content_settings_policy_provider.cc(279)] Skipping unset preference: profile.managed_cookies_allowed_for_urls
```

## 参考

- [https://www.chromium.org/for-testers/enable-logging/](https://www.chromium.org/for-testers/enable-logging/)
- [https://chromium.googlesource.com/chromium/src/+/HEAD/base/logging.h](https://chromium.googlesource.com/chromium/src/+/HEAD/base/logging.h)