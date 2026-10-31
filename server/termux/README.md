# Termux Server

Dokumentasi setup server Android menggunakan Termux.

## Device

- Device: Nubia Neo 2 5G
- OS: Android 13
- Environment: Termux
- Architecture: aarch64

## Server Environment

- Node.js: Backend runtime
- Python: Scripting
- Git: Version control
- OpenSSH: Remote access
- tmux: Persistent terminal sessions

## SSH Configuration

- SSH Server: OpenSSH
- Default Port: 8022
- Connection: ADB Port Forwarding
- Host: 127.0.0.1

## Directory Structure

- projects/: Source code aplikasi
- data/: Data aplikasi
- logs/: Log server
- backups/: Backup aplikasi

## Important Notes

- Termux digunakan sebagai lingkungan server.
- Server dijalankan pada perangkat Android.
- ADB forwarding digunakan untuk koneksi SSH melalui laptop.
- Jangan menyimpan password, token, atau private key di repository.
