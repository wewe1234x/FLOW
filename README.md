```mermaid
flowchart LR
    %% 定義兩個主要區塊
    subgraph Scaler[Scaler]
        S1[開始] --> S2{E0_C8 == 1?}
        S2 -- 是 不允许更新指令 --> S2
        S2 -- 否 可以更新指令--> S3[將 E0_DD = 0]
        S3 --> S4[寫指令到 E0_C0 ~ E0_C7]
        S4 --> S5[將 E0_DD = 1]
        S5 --> S6{指令類型?}
        S6 -- SET --> S7[等待 E0_C8 == 1]
        S7 --> S8[將 E0_DD = 0]
        S8 --> S9[SET 命令結束]
        S6 -- GET --> S10[等待 E0_C8 == 1]
        S10 --> S11[等待 E0_AE == 1]
        S11 --> S12[讀取 E0_A1 ~ E0_A3]
        S12 --> S13[將 E0_DD = 0]
        S13 --> S14[GET 命令結束]
    end

    subgraph Chip[芯片]
        C1[開始] --> C2{E0_DD == 1?}
        C2 -- 否 無Scaler指令 --> C2
        C2 -- 是 有Scaler指令--> C3[將 E0_C8 = 0]
        C3 --> C4[讀取 E0_C0 ~ E0_C7 並寫入 DPCD]
        C4 --> C5[將 E0_C8 = 1]
        C5 --> C6{指令類型?}
        C6 -- SET --> C7[等待 E0_DD == 0]
        C7 --> C8[將 E0_C8 = 0 復位]
        C8 --> C9[SET 命令結束]
        C6 -- GET --> C10[等待 R1 拉 HPD IRQ]
        C10 --> C11[讀取 DPCD 更新]
        C11 --> C12[將 E0_AE = 1]
        C12 --> C13[GET 命令結束]
    end

    %% 雙向交互箭頭
    S4 -.-> C4
    S5 -.-> C2
    S8 -.-> C7
    C12 -.-> S11
