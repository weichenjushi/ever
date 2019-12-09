技术002KLog

日志级别
========

journalctl -xefu kubelet的日志级别 –v=0 Generally useful for this to
ALWAYS be visible to an operator. –v=1 A reasonable default log level if
you don’t want verbosity.

–v=2 Useful steady state information about the service and important log
messages that may correlate to significant changes in the system. This
is the recommended default log level for most systems.

–v=3 Extended information about changes. –v=4 Debug level verbosity.
–v=6 Display requested resources. –v=7 Display HTTP request headers.
–v=8 Display HTTP request contents. –v=9 Display HTTP request contents
without truncation of contents.

日志搜集方案
============

-  云原生日志方案

https://juejin.im/post/5b6eaef96fb9a04fa25a0d37

-  小米

https://mp.weixin.qq.com/s/iBQzN3DtIPa3wZ96d5Uvng
Prometheus监控Kubernetes系列1——监控框架
https://mp.weixin.qq.com/s/rG1_DqjBjisuhQJNi9U7iA
Prometheus监控Kubernetes系列2——监控部署
https://mp.weixin.qq.com/s/l0yWzLMC4KFNkFwjtDZOeQ
Prometheus监控Kubernetes系列3——业务指标采集

%0A%0A%23%23%20%E6%97%A5%E5%BF%97%E7%BA%A7%E5%88%AB%0Ajournalctl%20-xefu%20kubelet%E7%9A%84%E6%97%A5%E5%BF%97%E7%BA%A7%E5%88%AB%0A–v%3D0%20%20%20%20Generally%20useful%20for%20this%20to%20ALWAYS%20be%20visible%20to%20an%20operator.%0A–v%3D1%20%20%20%20A%20reasonable%20default%20log%20level%20if%20you%20don%E2%80%99t%20want%20verbosity.%0A–v%3D2%20%20%20%20Useful%20steady%20state%20information%20about%20the%20service%20and%20important%20log%20messages%20that%20may%20correlate%20to%20significant%20changes%20in%20the%20system.%20This%20is%20the%20recommended%20default%20log%20level%20for%20most%20systems.%0A–v%3D3%20%20%20%20Extended%20information%20about%20changes.%0A–v%3D4%20%20%20%20Debug%20level%20verbosity.%0A–v%3D6%20%20%20%20Display%20requested%20resources.%0A–v%3D7%20%20%20%20Display%20HTTP%20request%20headers.%0A–v%3D8%20%20%20%20Display%20HTTP%20request%20contents.%0A–v%3D9%20%20%20%20Display%20HTTP%20request%20contents%20without%20truncation%20of%20contents.%0A%0A%23%23%20%E6%97%A5%E5%BF%97%E6%90%9C%E9%9B%86%E6%96%B9%E6%A1%88%0A-%20%E4%BA%91%E5%8E%9F%E7%94%9F%E6%97%A5%E5%BF%97%E6%96%B9%E6%A1%88%0Ahttps%3A%2F%2Fjuejin.im%2Fpost%2F5b6eaef96fb9a04fa25a0d37%0A-%20%E5%B0%8F%E7%B1%B3%0Ahttps%3A%2F%2Fmp.weixin.qq.com%2Fs%2FiBQzN3DtIPa3wZ96d5Uvng%0APrometheus%E7%9B%91%E6%8E%A7Kubernetes%E7%B3%BB%E5%88%971%E2%80%94%E2%80%94%E7%9B%91%E6%8E%A7%E6%A1%86%E6%9E%B6%0Ahttps%3A%2F%2Fmp.weixin.qq.com%2Fs%2FrG1_DqjBjisuhQJNi9U7iA%0APrometheus%E7%9B%91%E6%8E%A7Kubernetes%E7%B3%BB%E5%88%972%E2%80%94%E2%80%94%E7%9B%91%E6%8E%A7%E9%83%A8%E7%BD%B2%0Ahttps%3A%2F%2Fmp.weixin.qq.com%2Fs%2Fl0yWzLMC4KFNkFwjtDZOeQ%0APrometheus%E7%9B%91%E6%8E%A7Kubernetes%E7%B3%BB%E5%88%973%E2%80%94%E2%80%94%E4%B8%9A%E5%8A%A1%E6%8C%87%E6%A0%87%E9%87%87%E9%9B%86
