```mermaid
flowchart LR
    %% 定義兩個主要區塊
    subgraph Scaler[Scaler]
        S1[開始] --> S2{if E0_AC == 0?}
        S2 -- 否 不允许更新指令 --> S2
        S2 -- 是 可以更新指令--> S3[寫指令到 E0_C0 ~ E0_C7]
        S3 --> S4[set E0_B0 = 1]
        S4 --> S5{if E0_AC == 1?}
        S5 --> S6{指令類型?}
        S6 -- SET --> S7[set E0_B0 = 0]
        S6 -- GET -->S9[讀取 E0_A0 ~ E0_AB]
        S9 --> S7
        S7 --> S2
    end

    subgraph Chip[芯片]
        C1[開始] --> C2{E0_B0 == 1?}
        C2 -- 否 無Scaler指令 --> C2
        C2 -- 是 有Scaler指令--> C3[讀取 E0_C0 ~ E0_C7 並寫入 DPCD]
        C3 --> C4{指令類型?}
        C4 -- SET --> C5[完成傳送到R1]
        C5 --> C6[set E0_AC = 1 ]
        C6 --> C7{if E0_B0 == 0}
        C7 -- 否 --> C7
        C7 -- 是--> C8[set E0_AC = 0 ] 
        C4 -- GET --> C9[跟 R1 溝通完成 收到回傳資料 ]
        C9 --> C10[收到回傳資料寫到A0~AB]
        C10 --> C6
        C8 --> C2
    end

    %% 雙向交互箭頭
    S4 -.-> C2
    C6 -.-> S5
    C8 -.-> S2
    S7 -.->C7
