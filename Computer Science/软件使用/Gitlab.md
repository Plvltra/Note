# Ci/Cd
#### Runners
- GitLab Runner 是一个轻量级的、高扩展性的代理程序，用于执行 GitLab CI/CD 流水（Pipeline）中定义的作业（Jobs）。它负责运行自动化任务（如编译代码、运行测试、部署应用等），并将结果反馈给 GitLab 服务器。
- 在gitlab_runner register的时候, 生成一份配置config.toml, 用于定义 Runner 的全局设置、运行参数以及如何执行 CI/CD 作业
- 开发者推送代码到 GitLab 仓库，触发 `.gitlab-ci.yml` 文件中定义的流水线
#### Variables
- Variables: 用于存储动态值(如密码、API密钥、环境路径等)的键值对，可在 CI/CD 流水线中重复使用。它们帮助实现配置的灵活性和安全性。