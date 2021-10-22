Slack
===

Slack 的开源替代品 [Mattermost](https://github.com/mattermost)，使用下面命令即可启动 Mattermost，并且可以直接使用。

```bash
docker run --name mattermost-preview \
  --publish 8065:8065 \
  --add-host dockerhost:127.0.0.1 \
  --rm \
  -d mattermost/mattermost-preview:5.4.0
```

上面命令直接在命令行运行，就可以使用，通常情况我们想映射配置和存储，以便删除容器数据还存在，避免数据丢失。首先我们需要通过上面运行好的容器，将里面的配置拷贝出来。

```bash
docker container cp mattermost-preview:/mm/mattermost/config $HOME/_docker/mattermost/
```

挂载配置目录数据库目录运行

```bash
docker run --name mattermost-preview \
  --publish 8065:8065 \
  --add-host dockerhost:127.0.0.1 \
  -v $HOME/_docker/mattermost/config:/mm/mattermost/config \
  -v $HOME/_docker/mattermost/mysql:/var/lib/mysql \
  -v $HOME/_docker/mattermost/data:/mm/mattermost-data \
  -d mattermost/mattermost-preview:5.4.0
```

## 设置gitlab单点登陆

这个功能官方说需要购买 [Mattermost Enterprise Edition](https://about.mattermost.com/pricing/) 版本，文档这里: [GitLab Single Sign-On](https://docs.mattermost.com/deployment/sso-gitlab.html)。

修改 mattermost 配置 `vim $HOME/_docker/mattermost/config/config.json`。

```json
"GitLabSettings": {
  "Enable": true,
  "Secret": "{mattermost-app-secret-from-gitlab}",
  "Id": "{mattermost-app-application-id-from-gitlab}",
  "Scope": "",
  "AuthEndpoint": "https://{gitlab-site-name}/oauth/authorize",
  "TokenEndpoint": "https://{gitlab-site-name}/oauth/token",
  "UserApiEndpoint": "https://{gitlab-site-name}/api/v4/user"
}
```

[您可以在GitLab服务器上运行GitLab Mattermost服务。](https://docs.gitlab.com/omnibus/gitlab-mattermost/)，这篇文档介绍了修改 gitlab 配置。

```bash
vim /etc/gitlab/gitlab.rb

mattermost_external_url 'http://mattermost.example.com'

gitlab_rails['mattermost_host'] = "https://mattermost.example.com"
```

下面这部分配置在 `gitlab.rb` 中设置，之后**还需要**在 `mattermost` 系统中设置 [http://mattermost.example.com/admin_console/authentication/gitlab](http://mattermost.example.com/admin_console/authentication/gitlab)

```bash
mattermost['gitlab_enable'] = true
mattermost['gitlab_id'] = "12345656"
mattermost['gitlab_secret'] = "123456789"
mattermost['gitlab_scope'] = ""
mattermost['gitlab_auth_endpoint'] = "http://gitlab.example.com/oauth/authorize"
mattermost['gitlab_token_endpoint'] = "http://gitlab.example.com/oauth/token"
mattermost['gitlab_user_api_endpoint'] = "http://gitlab.example.com/api/v4/user"
```

如果您使用的是 `GitLab Mattermost`，请在系统控制台或 `gitlab.rb` 中[配置您的站点URL](https://docs.mattermost.com/administration/config-settings.html?highlight=add%20members%20team#site-url)。

在配置 `vim $HOME/_docker/mattermost/config/config.json`，这个目录是你挂载出来的配置目录文件，容器中实际目录 `/mm/mattermost/config`，修改下面内容。

```json
"ServiceSettings": {
  "SiteURL": "http://mattermost.example.com",
}
```
