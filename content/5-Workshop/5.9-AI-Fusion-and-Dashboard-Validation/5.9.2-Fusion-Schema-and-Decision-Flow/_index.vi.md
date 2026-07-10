---
title : "Fusion schema và decision flow"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.9.2. </b> "
---


## Mục tiêu

Ghi lại Fusion schema và cách model signals được chuyển thành final alert contract.

## Fusion schema

Fusion kết hợp AI outputs và security evidence thành một final alert object với các field như event ID, model results, severity, risk score, confidence score, reason và downstream storage target.

![Fusion schema](/images/5-Workshop/5.9-AI-Fusion-and-Dashboard-Validation/w-tin-21-fusion-schema.png)

## Decision flow

1. Nhận AI1 output và evidence hiện có.
2. Kiểm tra AI2A hoặc AI2B output có sẵn hay không.
3. Ghi rõ model output nào unavailable.
4. Kết hợp score và evidence thành một final alert.
5. Chuyển final alert cho RDS và downstream dashboard consumers.

## Final alert contract

Final alert cần đủ ổn định để lưu trữ và phục vụ dashboard, kể cả khi một số model output đang là mock hoặc unavailable.
