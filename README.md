# Ansibleに関するメモ
## 公式サイト
`http://docs.ansible.com/ansible/latest/index.html`

## 導入方法
yumでインストールできる  
`yum install ansible`

フォルダ構成は下記参照  
`http://docs.ansible.com/ansible/latest/playbooks_intro.html`

対象となるホストは下記に追加  
`/etc/ansible/hosts`  
```
[webservers]
192.168.11.10
192.168.11.11
```

ymlファイルを作成する  
#httpdをインストールする  
` /etc/ansible/test.yml`
```
- hosts: webservers
  remote_user: root
  tasks:
    - name: install httpd
      yum: name=httpd
           state=latest

    - name: write the apache config file
      template:
        src:  /etc/ansible/roles/webserv/templates/httpd.conf
        dest: /etc/httpd.conf

    - name: start service httpd
      service: name=httpd
               state=started
               enabled=yes
```

実行方法  
```
cd /etc/ansible
ansible-playbook -i hosts test.yml --ask-pass
オプション
-i : hostsファイルの指定
-u : ログインユーザの指定
--private-key ： 秘密鍵の指定
```

### 共通実行の例
format.yml
```
- hosts: all
  remote_user: root
  roles:
    - common

- hosts: webservers
  remote_user: root
  roles:
    - webserv
```

~/roles/common/tasks/main.yml
```
- name: add a new user      #ユーザ作成
  user: name=ansible
        state=present

- name: mkdir .ssh          #ディレクトリ作成
  file: dest=/home/ansible/.ssh/
        state=directory
        owner=ansible
        group=ansible
        mode=700

- name: add authorized keys #認証鍵のコピー
  file: dest=<公開鍵のキー>
        state=touch
        owner=ansible
        group=ansible
        mode=600
```

~/roles/webserv/tasks/main.yml
```
- name: install httpd
  yum: name=httpd
        state=latest

- name: write the apache config file
  template:
    src:  /etc/ansible/roles/webserv/templates/httpd.conf
    dest: /etc/httpd.conf

- name: start service httpd
  service: name=httpd
           state=started
           enabled=yes
```

### エラーの例
一度も該当ホストにssh-loginしていない場合に発生する
```
TASK [Gathering Facts] **************************************************************

fatal: [192.168.11.12]: FAILED! => {"msg": "Using a SSH password instead of a key is not possible because Host Key checking is enabled and sshpass does not support this. 
Please add this host's fingerprint to your known_hosts file to manage this host."}
```
< 解決方法 >  
`/etc/ansible/ansible.cfg` に下記設定を追記
```
[ssh_connection]
ssh_args = -C -o ControlMaster=auto -o ControlPersist=60s -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null
```

