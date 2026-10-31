

## Overview

SSH digunakan untuk mengakses terminal Termux
dari laptop tanpa harus berinteraksi langsung
dengan layar HP.

## Connection Architecture

Laptop
  |
  | ADB Port Forwarding
  |
127.0.0.1:8022
  |
  | SSH
  |
Termux SSH Server

## ADB Forwarding

```bash
adb forward tcp:8022 tcp:8022


ssh -p 8022 USERNAME@127.0.0.1
