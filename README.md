# n8n Telegram Webhook - Rozwiązanie Hybrydowe HTTPS/HTTP

## 🎯 Podsumowanie problemu i rozwiązania

**Problem:** Telegram Bot API wymaga HTTPS URL dla webhook, ale n8n na serwerze Mikrus generował tylko HTTP URL.

**Rozwiązanie:** Konfiguracja hybrydowa gdzie:
- **Główny n8n**: `http://joanna358.mikrus.xyz:20358` (OAuth, interfejs, API)  
- **Webhook Telegram**: `https://[cloudflare-tunnel]` (tylko dla Telegram)

## 🏗️ Architektura rozwiązania

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Telegram      │    │  Cloudflare      │    │   Serwer        │
│   Bot API       │────│  Tunnel (HTTPS)  │────│   Mikrus        │
│                 │    │                  │    │   n8n:20358     │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                                                         │
┌─────────────────┐                                     │
│   OAuth/Google  │─────────────────────────────────────┘
│   Calendar/Gmail│           HTTP (stabilny URL)
└─────────────────┘
```

## 📋 Aktualna konfiguracja (stan: 2 czerwca 2025)

### Serwer Mikrus
- **Host**: `joanna358.mikrus.xyz`
- **Port SSH**: `10358`  
- **Port n8n**: `20358` (mapowany na 5678 w kontenerze)
- **Dostęp SSH**: `ssh -i ~/.ssh/id_ed25519 -p 10358 root@joanna358.mikrus.xyz`

### n8n
- **URL główny**: `http://joanna358.mikrus.xyz:20358`
- **Wersja**: `1.90.2`
- **Kontener**: `n8nio/n8n:latest`
- **Volume**: `~/.n8n:/home/node/.n8n`

### Cloudflare Tunnel  
- **Aktualny URL**: `https://prototype-ds-constitute-efforts.trycloudflare.com`
- **Status**: Tymczasowy (może się zmienić po restarcie)
- **Log file**: `/tmp/cloudflared.log`
- **PID file**: `/tmp/cloudflared.pid`

## 🚀 Szybki start - co zrobić TERAZ

### 1. Sprawdź aktualny status
```bash
# Sprawdź czy n8n działa
ssh -i ~/.ssh/id_ed25519 -p 10358 root@joanna358.mikrus.xyz "docker ps | grep n8n"

# Sprawdź tunnel
ssh -i ~/.ssh/id_ed25519 -p 10358 root@joanna358.mikrus.xyz "ps aux | grep cloudflared"
```

### 2. Sprawdź aktualny URL tunelu
```bash
ssh -i ~/.ssh/id_ed25519 -p 10358 root@joanna358.mikrus.xyz "grep 'trycloudflare.com' /tmp/cloudflared.log | tail -1"
```

### 3. Sprawdź webhook w n8n
1. Idź do: `http://joanna358.mikrus.xyz:20358`
2. Otwórz workflow z Telegram Trigger
3. Webhook URL powinien być: `https://[cloudflare-tunnel]/webhook/[hash]`

### 4. Jeśli webhook pokazuje localhost:5678
**To znaczy że zmienne środowiskowe nie są ustawione!** 

Uruchom pełną procedurę restartu - zobacz [Instrukcje techniczne](docs/instrukcje_techniczne.md).

## 📚 Dokumentacja

- **[Instrukcje techniczne](docs/instrukcje_techniczne.md)** - Szczegółowe komendy Docker, Cloudflare, Telegram
- **[Rozwiązywanie problemów](docs/rozwiazywanie_problemow.md)** - Najczęstsze problemy i naprawy
- **[Przykłady użycia](docs/przyklady_uzycia.md)** - Scenariusze i procedury operacyjne
- **[FAQ](docs/faq.md)** - Odpowiedzi na najczęstsze pytania

## 🛠️ Skrypty automatyzacji

```bash
# Walidacja konfiguracji
./scripts/validate_config.sh

# Backup danych
./scripts/backup_config.sh

# Przywracanie z backup
./scripts/restore_config.sh backup_file.tar.gz
```

## ⚠️ WAŻNE zasady bezpieczeństwa

### ✅ Co jest bezpieczne
- **Cloudflare tunnel**: Działa tylko w jedną stronę (outbound)
- **Szyfrowanie**: End-to-end szyfrowanie wszystkich danych  
- **Izolacja**: Tunnel pokazuje tylko webhook endpoint
- **Kontrola**: Można natychmiast wyłączyć tunnel

### 🚫 Czego NIE robić
- **Nie zmieniać OAuth URL na HTTPS** - OAuth wymaga stabilnego URL
- **Nie ustawiać N8N_PORT** - powoduje konflikty portów
- **Nie używać tunelu do OAuth** - tunnel ma niestabilne URL

## 🔧 Szybkie komendy

```bash
# Status systemu
curl -I http://joanna358.mikrus.xyz:20358
curl -I https://prototype-ds-constitute-efforts.trycloudflare.com

# Sprawdź webhook w Telegram (zamień [TOKEN])
curl -s "https://api.telegram.org/bot[TOKEN]/getWebhookInfo" | python3 -m json.tool
```

## 🆘 Emergency restart

Jeśli nic nie działa:

```bash
# 1. Zatrzymaj wszystko
ssh -i ~/.ssh/id_ed25519 -p 10358 root@joanna358.mikrus.xyz "docker stop n8n && docker rm n8n && pkill cloudflared"

# 2. Uruchom tunnel (nowy URL)
ssh -i ~/.ssh/id_ed25519 -p 10358 root@joanna358.mikrus.xyz "nohup cloudflared tunnel --url http://127.0.0.1:20358 > /tmp/cloudflared.log 2>&1 & sleep 15"

# 3. Sprawdź nowy URL i uruchom n8n z nowym URL
# Zobacz pełną procedurę w dokumentacji
```

## 🔗 Przydatne linki

- **n8n Interface**: `http://joanna358.mikrus.xyz:20358`
- **Aktualny tunnel**: `https://prototype-ds-constitute-efforts.trycloudflare.com`
- **MCP API**: Zintegrowane z Claude Desktop (759 workflow dostępnych)
- **GitHub Issues**: [Zgłoś problem](https://github.com/3rzy/n8n-telegram-webhook-guide/issues)

---
*Ostatnia aktualizacja: 2 czerwca 2025*  
*Status: Konfiguracja hybrydowa działa i została przetestowana*
