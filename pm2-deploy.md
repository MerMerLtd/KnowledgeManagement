## 使用pm2自動化部署

### 1. 把遠端服務器進行身份驗證的公鑰放到 ~/.ssh 資料夾內
#### 1. 建立.ssh file 並跳轉進入.ssh file
```
cd ..
mkdir ~/.ssh
cd ~/.ssh
``` 
#### 2. 把進行身份驗證的公鑰放到 ~/.ssh 資料夾內
```
mv /origin/path/to/some.pem /destination/path/to/some.pem
```

### 2. 使用npm全局安裝pm2

```
npm install pm2 -g
```

### 3. 使用pm2 自動生成部署檔案 ecosystem.config.js

打開terminal跳轉到要部署的app的檔案夾在package.json同一folder run以下指令
```
pm2 ecosystem
```

### 4. 根據要部署的環境編輯 ecosystem.config.js

```
module.exports = {
  // app 部份
  apps : [{
    name: 'API',
    // 入口檔案，以mermer-framework為例
    script: 'bin/main.js', 

    // Options reference: https://pm2.keymetrics.io/docs/usage/application-declaration/
    // args: 'one two', // 我們不需要傳入參數
    instances: 1,
    autorestart: true,
    watch: false,
    max_memory_restart: '1G',

    env: {
      NODE_ENV: 'development'
    },
   // --env production 開始時注入的環境變量
    env_production: {
      NODE_ENV: 'production'
    }
  }],

  // 部署部份
  deploy : {
    // 發佈階段部署的環境
    production : {
      // 使用者名稱
      user : 'ubuntu',
      // 要部署到的主機的IP地址
      key  : `${process.env.HOME}/.ssh/[Key].pem`, // 進行身份驗證的公鑰的路徑
      host : '54.238.248.160', // 也可以通過將IPs /主機名以arry傳入來實現多主機部署
      // branch （分支）
      ref  : 'origin/master',
      // 要 clone 的 Git repository
      repo : 'git@github.com:[repo].git',
      // 目標服務器上應用程序的路徑
      path : '/etc/[reponame]',
      // cloned後要在服務器上執行的命令
      'post-deploy' : 'npm install && pm2 reload ecosystem.config.js --env production',
      // 必須在此env的所有應用程序中註入的環境變量(我沒有加)
      env : {
        "NODE_ENV": "production"
      }
    },
    // 開發階段部署的環境
    staging : {
      user : "node",
      host : "212.83.163.1",
      ref : "origin/master",
      repo : "git@github.com:repo.git",
      path : "/etc/[reponame]",
      "post-deploy" : "pm2 startOrRestart ecosystem.config.js --env dev",
      env  : {
        "NODE_ENV": "staging"
      }
    }
  }
};
```
修改好後要更新到github上再進行下一步

### 4. 遠端連線要部署到的服務器
```
ssh -i some.pem ubuntu@54.238.248.160
```

如果出現 cannot create directory '/etc/[reponame]': Permission denied 的錯誤提示，則利用ssh連線到遠端服務區，到要建立檔案的folder（ex: etc/ )使用sudo 創建檔案，再把檔案所有權pass給User（ex: ubuntu）
```
ssh -i [Key].pem ubuntu@[IP address]
cd /etc
sudo mkdir [reponame]
sudo chown -R [user] [reponame]
```

### 5. 生成一對鑰匙，把公鑰放到git repository裡面確保對要clone的git repository有權限
```
cd ~/.ssh // 沒有的話就建立一個
ssh-keygen
Enter file in which to save the key (/Users/emilyliang/.ssh/id_rsa): [自訂]/默認為id_rsa
Enter passphrase (empty for no passphrase): [可填可不填]
vi [自訂/id_rsa].pub (複製貼上到git repository)
```
### 6. 使用以下命令初始化遠程文件夾
在terminal跳轉到要部署app的configuration_file所在的folder執行以下指令
```
pm2 deploy <configuration_file> <environment> setup
```
例如：
```
pm2 deploy ecosystem.config.js production setup
```

### 7. 部署代碼
```
pm2 deploy ecosystem.config.js production
```

Now your code will be populated, installed and started with PM2.
