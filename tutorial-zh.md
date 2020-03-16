# 使用 Ansible 管理云资源

## 简介

[Ansible](https://www.ansible.com) 是一个开源的用于自动执行资源的配置管理和应用程序部署产品。云命令行已经为您安装、配置完成 Ansible v2.8.5，您可以直接使用。同时阿里云提供了 Ansible 的 [阿里云模块](https://github.com/alibaba/ansible-provider)。

利用该该模块您可以通过 Ansible 非常方便管理您的阿里云资源，同时，您还可以使用 Ansible 在环境中自动配置资源和部署应用。

本教程介绍了如何使用 Ansible 来管理阿里云资源以及部署应用。

<tutorial-nav></tutorial-nav>

## 使用须知

### 计费

本教程默认帮您创建的资源有：

- ECS
    - 实例规格：ecs.t5-lc2m1.nano
    - 实例数量：2
    - 系统盘：20G 高效云盘

具体计费信息，参见 [ECS 计费概述](https://help.aliyun.com/document_detail/25398.html)。

## 配置 Ansible

在 Cloud Shell 中，您可以直接使用 Ansible 来管理您的云资源。Cloud Shell 为您内置了授权，并且默认配置好了阿里云模块，您无需进行额外配置。默认 Cloud Shell 会使用临时 AK 帮您配置好 AK 信息，同时默认 region 为 `cn-hangzhou`。

您也可以如下方式自行配置阿里云模块。

```
export ALICLOUD_ACCESS_KEY="your_accesskey"
```
```
export ALICLOUD_SECRET_KEY="your_accesskey_secret"
```
```
export ALICLOUD_REGION="your_region"
```

## 创建资源

在示例中，您会通过 Ansible 编排创建 VPC、VSwitch、安全组和 ECS。其中 `create.yml` playbook 声明了编排配置信息，<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/create.yml">在 Cloud Shell 编辑器中查看 create.yml</tutorial-editor-open-file>。其中，各资源的详细的配置分别在：

- 变量定义：<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/group_vars/all">group_vars/all</tutorial-editor-open-file>
- VPC：<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/roles/vpc/tasks/main.yml">roles/vpc/tasks/main.yml</tutorial-editor-open-file>
- VSwitch：<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/roles/vswitch/tasks/main.yml">roles/vswitch/tasks/main.yml</tutorial-editor-open-file>
- 安全组：<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/roles/security_group/tasks/main.yml">roles/security_group/tasks/main.yml</tutorial-editor-open-file>
- ECS：<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/roles/ecs/tasks/main.yml">roles/ecs/tasks/main.yml</tutorial-editor-open-file>

首先定位到 Ansible playbook 的目录。

```bash
cd ~/tutorial-cli-ansible/ansible
```

最后，您就可以一键配置好您编排的资源。

```bash
ansible-playbook create.yml
```

执行完后，您可以在对应产品的控制台看到编排创建的资源。比如该示例中默认为您创建了两台 ECS：

![](https://img.alicdn.com/tfs/TB1qUDIjhD1gK0jSZFsXXbldVXa-2386-412.png)

如果出现错误，请检查您的所需服务是否开通，以及您的账户是否实名认证同时账户余额大于 100 元。

## 动态 Inventory

当您需要配置资源、部署应用的时候，就需要配置 Ansible Inventory。Ansible 可同时操作属于一个组的多台主机。组和主机之间的关系通过 Inventory 文件配置。Ansible Inventory 分为静态 Inventory 和动态 Inventory。当被管理主机比较少的情况下，直接在静态 Inventory 的 host 文件中管理即可；当主机越来越多，不断变化时，可以通过动态 Inventory 来管理。

阿里云提供了动态 Inventory，您可以直接使用，通过阿里云动态 Inventory 动态获取指定过滤条件的主机信息。

### 环境准备

目前阿里云的动态 Inventory 还在被官方集成的过程中，您需要手动安装依赖。

```bash
pip install ansible_alicloud_module_utils
```

同时，升级 footmark，保证使用最新版本 footmark

```bash
pip install footmark --upgrade
```

### 下载与验证

首先您需要下载阿里云动态 Inventory 文件，并赋予其可执行权限

```bash
wget -P ~/tutorial-cli-ansible/ansible/ https://raw.githubusercontent.com/alibaba/ansible-provider/master/contrib/inventory/alicloud.py;\
chmod +x ~/tutorial-cli-ansible/ansible/alicloud.py
```

您可以执行 Inventory 文件来验证配置。

```bash
~/tutorial-cli-ansible/ansible/alicloud.py --list
```

执行完后，会返回所有地域的 Inventory 信息。其中对应的配置文件为 <tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/alicloud.ini">alicloud.ini</tutorial-editor-open-file>，你可以编辑 <tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/alicloud.ini">alicloud.ini</tutorial-editor-open-file> 文件获取指定 Region 的信息。除此之外，还可通过 <tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/alicloud.ini">alicloud.ini</tutorial-editor-open-file> 文件中的 instance_filters 过滤所有的主机信息。

您可以通过 Ansible ping 模块来验证和您创建的两台 ECS 实例的连通性

```bash
ansible -i ~/tutorial-cli-ansible/ansible/alicloud.py tag_created_from_cloudshell_tutorial_cli_ansible -m ping -u root -k
```

其中 `tag_created_from_cloudshell_tutorial_cli_ansible` 表示我们根据 `created_from_cloudshell_tutorial_cli_ansible` 的 tag 的过滤 ecs。执行命令后提示您输入 SSH 密码，编排资源时，示例中默认设置的密码为：`Test12345`。结果如下所示：

![](https://img.alicdn.com/tfs/TB16qHTjpP7gK0jSZFjXXc5aXXa-2112-1054.png)

## 部署应用

通过 Ansible 您可以配置资源并部署应用，本教程会部署 Nginx 到前文您创建的 ECS 中。`deploy.yml` playbook 声明了部署配置信息，<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/deploy.yml">在 Cloud Shell 编辑器中查看 deploy.yml</tutorial-editor-open-file>。其中配置 `hosts: tag_created_from_cloudshell_tutorial_cli_ansible`，根据 `created_from_cloudshell_tutorial_cli_ansible` 的 tag 来过滤 ecs。执行命令后提示您输入 SSH 密码，编排资源时，示例中默认设置的密码为：`Test12345`。

```bash
ansible-playbook -i ~/tutorial-cli-ansible/ansible/alicloud.py ~/tutorial-cli-ansible/ansible/deploy.yml -u root -k
```

执行结果如下所示：

![](https://img.alicdn.com/tfs/TB1u8Y.jxn1gK0jSZKPXXXvUXXa-2848-1054.png)

执行完后，Nginx 已经成功在您的 ECS 实例上安装并启动，其实你可以访问该 ECS 的公网 IP 查看。

![](https://img.alicdn.com/tfs/TB1DMMcjrj1gK0jSZFuXXcrHpXa-2874-840.png)

## 销毁资源

通过 Ansible 您也可以销毁资源。其中 `destory.yml` playbook 声明了销毁资源的配置信息，<tutorial-editor-open-file filePath="tutorial-cli-ansible/ansible/destory.yml">在 Cloud Shell 编辑器中查看 destory.yml</tutorial-editor-open-file>。

您可以执行以下命令来销毁前文创建的资源。

```bash
ansible-playbook ~/tutorial-cli-ansible/ansible/destory.yml
```

## 总结

恭喜！

您已完成学习如何使用 Ansible 管理阿里云资源和部署应用！