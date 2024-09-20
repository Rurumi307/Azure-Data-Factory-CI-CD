# 資料同步管理

[[_TOC_]]

## 說明

Budget System 需要實際資料來輔助預估 Budget, 而各實際資料存在不同系統內, 各系統於每年/月將實際資料上傳至中介資料庫, 再透過 Hyperion Batch 將資料同步至 Budget System.

## 資料同步機制

為讓資料同步流程可以有效管理及設定排程, 本系統是使用 Azure Data Factory (ADF) 來執行資料同步作業. 每個資料源都會建立一個 Pipeline 來同步資料.

![architecture][etl-architecture]

## 資料源同步清單

### P&L 實際數

**資料表名稱:** `PNL_ACTUAL`

**歷史資料表名稱:** `PLN_HISTORY_MONTHLY`

**對應介接批次:** `DL_A_PL.bat`

**ADF Pipeline:** `??`

#### EBG

每月資料上傳

歷史資料上傳

- 設定變數

  ```shell
  vault_name=ebg-adf-qas-vault
  api_host=ebg-api-qas.inventec.com
  subscription=EBG_IT_Efficiency
  ```

- 由 key vault 取得 Client Application 設定

  ```shell
  az account set --subscription $subscription \
  && client_id=$(az keyvault secret show --name budget-system-api-client-id --vault-name $vault_name | jq .value | sed 's/"//g') \
  && client_scope=$(az keyvault secret show --name budget-system-api-client-scope --vault-name $vault_name | jq .value | sed 's/"//g') \
  && client_secret=$(az keyvault secret show --name budget-system-api-client-secret --vault-name $vault_name | jq .value | sed 's/"//g')
  ```

- 取得 access token

  ```shell
  token=$(curl --request POST --url https://login.microsoftonline.com/2ae41f0c-acca-40f1-9c63-49475ff38512/oauth2/v2.0/token \
  --header 'Content-Type: multipart/form-data' \
  --form "client_id=$client_id" \
  --form "client_secret=$client_secret" \
  --form "grant_type=client_credentials" \
  --form "scope=$client_scope" \
  | jq .access_token | sed 's/"//g')
  ```

- 呼叫 API 將資料上傳至中介資料庫

  ```shell
  curl -v --location "https://$api_host/ebg-budget-system-api/api/ProfitAndLosts/actuals/months" \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer $token" \
  --data '{
    "months": [
        "202201",
        "202202",
        "202203",
        "202204",
        "202205",
        "202206",
        "202207",
        "202208",
        "202209",
        "202210",
        "202211",
        "202212"
    ]
  }'
  ```

### Product Sales Plan&銷量實際數

### Build Plan 實際數

### 月平均匯率

**資料表名稱:** `EXCHANGE_RATE`

**歷史資料表名稱:** `PLN_HISTORY_MONTHLY` `FA_HISTORY_MONTHLY`

**對應介接批次:** `DL_A_FX_IECPLN.bat` `DL_A_FX_IECFA.bat".bat`

**ADF Pipeline:** `??`

每月資料上傳

- 執行 ADF Pipeline

歷史資料上傳

- 設定變數

  ```shell
  vault_name=ebg-adf-qas-vault
  api_host=ebg-api-qas.inventec.com
  subscription=EBG_IT_Efficiency
  ```

- 由 key vault 取得 Client Application 設定

  ```shell
  az account set --subscription $subscription \
  && client_id=$(az keyvault secret show --name budget-system-api-client-id --vault-name $vault_name | jq .value | sed 's/"//g') \
  && client_scope=$(az keyvault secret show --name budget-system-api-client-scope --vault-name $vault_name | jq .value | sed 's/"//g') \
  && client_secret=$(az keyvault secret show --name budget-system-api-client-secret --vault-name $vault_name | jq .value | sed 's/"//g')
  ```

- 取得 access token

  ```shell
  token=$(curl --request POST --url https://login.microsoftonline.com/2ae41f0c-acca-40f1-9c63-49475ff38512/oauth2/v2.0/token \
  --header 'Content-Type: multipart/form-data' \
  --form "client_id=$client_id" \
  --form "client_secret=$client_secret" \
  --form "grant_type=client_credentials" \
  --form "scope=$client_scope" \
  | jq .access_token | sed 's/"//g')
  ```

- 呼叫 API 將資料上傳至中介資料庫

  ```shell
  curl -v --location "https://$api_host/ebg-budget-system-api/api/ExchangeRates/actuals/months" \
  --header 'Content-Type: application/json' \
  --header "Authorization: Bearer $token" \
  --data '{
    "months": [
        "202201",
        "202202",
        "202203",
        "202204",
        "202205",
        "202206",
        "202207",
        "202208",
        "202209",
        "202210",
        "202211",
        "202212"
    ]
  }'
  ```

### 費用會科實際數

### 現有資產折舊數 (預算年度預算數)

### 現有資產折舊數 (當年度 Budget Rolling)

### 預算編號 Rolling

### 預算編號 編列預算前

### Open PR 預算數 Rolling

### FA 預算資訊 Rolling

### FA 預算數 Rolling

### FA 實際數

### Headcount 實際數

### Budget Product Select

### IECHC Entity 本期新增項目

### IECPLN Entity 本期新增項目

[etl-architecture]: ./assets/budget-system-etl-architecture.png
