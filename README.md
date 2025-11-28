# cloudfile
用来保存wps文件
文件分为网络规划师学习文档。和我自己的学习心路

mermaid

graph TD
    %% 定义样式
    classDef internet fill:#e1f5fe,stroke:#01579b,stroke-width:2px;
    classDef firewall fill:#ffccbc,stroke:#bf360c,stroke-width:2px;
    classDef core fill:#fff9c4,stroke:#fbc02d,stroke-width:2px;
    classDef access fill:#dcedc8,stroke:#33691e,stroke-width:2px;
    classDef device fill:#f5f5f5,stroke:#9e9e9e,stroke-width:1px,stroke-dasharray: 5 5;

    %% 节点定义
    ISP((互联网 Internet)):::internet
    
    FW[("出口防火墙 (FW)<br/>Gateway<br/>NAT/VPN")]:::firewall
    
    CORE["核心交换机 (Core-SW)<br/>Layer 3 Routing<br/>DHCP Server<br/>Gateway for VLANs"]:::core
    
    ACC_OFFICE["接入交换机 A (Office)<br/>Layer 2"]:::access
    ACC_SERVER["接入交换机 B (Server)<br/>Layer 2"]:::access
    ACC_WIFI["PoE 交换机 C (Wireless)<br/>Layer 2 / PoE"]:::access

    %% 终端设备
    PC_STAFF["员工 PC<br/>IP: DHCP"]:::device
    SRV_AD["文件/AD 服务器<br/>IP: 10.0.20.10"]:::device
    AP_WIFI["无线 AP (Wi-Fi 6)<br/>Mgmt IP: 10.0.10.x"]:::device
    PC_GUEST["访客手机<br/>(Wi-Fi SSID: Guest)"]:::device

    %% 连接关系与接口配置
    ISP -- "WAN口 (PPPoE/Static)" --> FW
    
    FW -- "Ge0/0/1<br/>IP: 172.16.1.1/30" --> CORE
    
    CORE -- "Ge0/0/24<br/>Mode: Route<br/>IP: 172.16.1.2/30" --> FW

    %% 核心向下联接
    CORE -- "Ge0/0/1<br/>Trunk<br/>Allow: VLAN 30" --> ACC_OFFICE
    CORE -- "Ge0/0/2<br/>Trunk<br/>Allow: VLAN 20" --> ACC_SERVER
    CORE -- "Ge0/0/3<br/>Trunk<br/>Allow: VLAN 10,40,50" --> ACC_WIFI

    %% 接入层连接终端
    ACC_OFFICE -- "Ge0/0/1<br/>Access<br/>Default VLAN 30" --> PC_STAFF
    ACC_SERVER -- "Ge0/0/1<br/>Access<br/>Default VLAN 20" --> SRV_AD
    ACC_WIFI -- "Ge0/0/1<br/>Trunk<br/>PVID 10 (Mgmt)" --> AP_WIFI
    
    %% 逻辑映射
    AP_WIFI -.->|"SSID: Staff<br/>(VLAN 40)"| CORE
    AP_WIFI -.->|"SSID: Guest<br/>(VLAN 50)"| PC_GUEST

    %% 子图标注
    subgraph VLAN_PLAN [VLAN 规划表]
        direction LR
        v10[VLAN 10: Mgmt<br/>10.0.10.1/24]
        v20[VLAN 20: Servers<br/>10.0.20.1/24]
        v30[VLAN 30: Staff<br/>10.0.30.1/24]
        v40[VLAN 40: WiFi-Int<br/>10.0.40.1/24]
        v50[VLAN 50: Guest<br/>192.168.50.1/24]
    end