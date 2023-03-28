## Self-hosted Github Action 如何实现

可以查看[在 Kubernetes 上执行 GitHub Actions 流水线作业](https://atbug.com/run-github-actions-runners-on-kubernetes/)实现在K8S上执行Github Actions，这同样可以跑在阿里云ASK等虚拟K8S集群上。

下面这篇文章阐述了如何进行成本优化：[GitHub Actions 成本优化：让你的团队更具竞争力](https://moelove.info/2023/03/21/GitHub-Actions-%E6%88%90%E6%9C%AC%E4%BC%98%E5%8C%96%E8%AE%A9%E4%BD%A0%E7%9A%84%E5%9B%A2%E9%98%9F%E6%9B%B4%E5%85%B7%E7%AB%9E%E4%BA%89%E5%8A%9B/)。

 - Github Action的Linux/Ubuntu(2 Cores and 8GB memory)环境是0.008刀每分钟（来自[官方文档](https://docs.github.com/zh/billing/managing-billing-for-github-actions/about-billing-for-github-actions)），大约是0.055元人民币每分钟。
 - 若使用阿里云ASK，同样的规格Linux/Ubuntu(2 Cores and 8GB memory) 是 (0.000049*2+0.00000613*8)*60=0.0088元人民币每分钟，仅为Github Actions官网成本的 1/5，缺点是要部署折腾一下。