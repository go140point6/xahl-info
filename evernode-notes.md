Check cert of reputationd

```
cd /etc/sashimono/contract_template/cfg/
sudo openssl x509 -enddate -noout -in tlscert.pem
```

```
sudo evernode applyssl /etc/letsencrypt/live/<your host domain>/privkey.pem /etc/letsencrypt/live/<your host domain>/cert.pem /etc/letsencrypt/live/<your host domain>/fullchain.pem
```

Many reports of momements without scores

```
sudo -u sashireputationd bash -c journalctl --user -u sashimono-reputationd | grep "Reporting"
```

Reputationd logs

```
sudo -u sashireputationd bash -c 'journalctl --user -u sashimono-reputationd | tail -n 200'
```
