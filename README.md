1. AWS EC2 部署與 Key Pair 管理  
2. Ansible Playbook 與 Role 實作

---

## AWS 自動化部署功能

- **檢查 Key Pair 是否存在**  
  避免重複建立 key，確保安全性。  

- **建立 Key Pair（如果不存在）**  
  首次建立時，自動下載私鑰到本地。  

- **安全保存私鑰**  
  設定檔案權限為 `600`，確保只有擁有者可讀寫。  

- **建立 EC2 instance**  
  - 指定 AMI、instance類型、名稱與標籤  
  - 等待instance啟動完成  

---

## Playbook 與 Role 演進

### playbook – 單一 Playbook
- 使用單一 playbook (`provisioning.yaml`) 管理所有任務  
- 任務內容：
  - 安裝 NTP/Chrony
  - 啟動服務
  - 部署模板
  - 建立資料夾
- 展示 **Ansible 變數優先權**：
  - `host_vars`
  - `group_vars`
  - Playbook 變數

### role – Role 化
```roles/
└── post-install/                # Role 名稱
    ├── defaults/                # 預設變數（最低優先權）
    │   └── main.yml
    │
    ├── vars/                    # 角色變數（較高優先權）
    │   └── main.yml
    │
    ├── tasks/                   # 主要任務定義
    │   └── main.yml
    │
    ├── handlers/                # 被 notify 觸發的任務（重啟服務等）
    │   └── main.yml
    │
    ├── templates/               # Jinja2 模板（.j2）
    │   ├── ntpconf_centos.j2
    │   └── ntpconf_ubuntu.j2
    │
    ├── files/                   # 靜態檔案（直接複製）
    │   └── myfile.txt
    │
    ├── meta/                    # 角色中繼資料（Galaxy / 相依性）
    │   └── main.yml
    │
    ├── tests/                   # Role 測試用
    │   ├── inventory
    │   └── test.yml
    │
    └── README.md                # Role 說明文件```

- 將 playbook directory 的內容轉為 `post-install` Role  
- 封裝安裝、配置、模板部署、服務管理、資料夾建立等任務  
- 使用 **角色結構**：
  - `defaults`、`vars`、`tasks`、`handlers`、`templates`、`files`  
- Playbook (`provisioning.yaml`) 只需引用 Role 即可執行全部任務  
