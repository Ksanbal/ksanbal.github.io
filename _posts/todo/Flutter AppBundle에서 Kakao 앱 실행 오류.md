- flutter에서 직접 빌드하면 이상이 없는데
- play store로 업로드 되면 이슈가 생김
- sha-1을 카카오 개발자쪽에 빼먹은게 있었
```zsh
echo {sha-1} | xxd -r -p | openssl base64
```

