# 我的 AWS 架構圖

```mermaid
graph LR
    %% --- AWS 官方配色樣式定義 ---
    classDef compute fill:#FF9900,stroke:#232F3E,color:white,font-weight:bold,rx:5,ry:5;
    classDef storage fill:#3F8624,stroke:#232F3E,color:white,font-weight:bold,rx:5,ry:5,shape:cylinder;
    classDef integration fill:#E7157B,stroke:#232F3E,color:white,font-weight:bold,rx:5,ry:5;
    classDef management fill:#CC2264,stroke:#232F3E,color:white,font-weight:bold,rx:5,ry:5;
    classDef external fill:#FFFFFF,stroke:#545B64,color:#545B64,stroke-dasharray: 5 5,stroke-width:2px,rx:5,ry:5;

    %% --- 節點定義 ---
    User((使用者/User))
    WeatherAPI[氣象署 API]:::external
    LineAPI[Line Notify API]:::external

    subgraph AWS["AWS Cloud (Serverless Backend)"]
        direction LR
        
        %% 1. 觸發 (Integration)
        EventBridge[Amazon EventBridge<br/>(排程觸發)]:::integration
        
        %% 2. 收集 (Compute & Storage)
        LambdaCollect[AWS Lambda<br/>(Collector)]:::compute
        %% 使用圓柱體形狀表示儲存
        S3Raw[(Amazon S3<br/>Raw Data)]:::storage
        
        %% 3. 分析與通知 (Compute, Management, Storage, Integration)
        LambdaAnalyze[AWS Lambda<br/>(Analyzer)]:::compute
        SSM[Systems Manager<br/>Parameter Store]:::management
        S3Result[(Amazon S3<br/>Result Image)]:::storage
        SNS[Amazon SNS<br/>(Topic)]:::integration
    end

    %% --- 連線 ---
    EventBridge -- "1" --> LambdaCollect
    LambdaCollect -- "2" --> WeatherAPI
    WeatherAPI -.-> LambdaCollect
    LambdaCollect -- "3" --> S3Raw
    
    S3Raw -- "4 (Trigger)" --> LambdaAnalyze
    
    LambdaAnalyze -- "5" --> SSM
    SSM -.-> LambdaAnalyze
    
    LambdaAnalyze -- "6 (Alert)" --> LineAPI
    LineAPI -- "7" --> User
    
    LambdaAnalyze -- "8" --> S3Result
    LambdaAnalyze -- "9" --> SNS
    SNS -- "10" --> User
```
