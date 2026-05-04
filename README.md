Установка 

#!/bin/bash
set -e

BOT_DIR="/opt/iiipki_robot"
SERVICE_NAME="iiipki_robot"
BOT_USER="root"

echo "==> Установка пакетов..."
apt update
apt install -y python3 python3-venv python3-pip ffmpeg curl git

echo "==> Создание папки бота..."
mkdir -p "$BOT_DIR"
mkdir -p "$BOT_DIR/downloads"

echo "==> Создание venv..."
cd "$BOT_DIR"
python3 -m venv venv

echo "==> Установка зависимостей..."
"$BOT_DIR/venv/bin/pip" install --upgrade pip
"$BOT_DIR/venv/bin/pip" install aiogram python-dotenv yt-dlp

echo "==> Создание systemd сервиса..."
cat > /etc/systemd/system/${SERVICE_NAME}.service <<EOF
[Unit]
Description=iiipki Telegram Bot
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=${BOT_USER}
WorkingDirectory=${BOT_DIR}
ExecStart=${BOT_DIR}/venv/bin/python ${BOT_DIR}/bot.py
Restart=always
RestartSec=5

# лимиты (чтобы ffmpeg не убивал сервер)
LimitNOFILE=65535

[Install]
WantedBy=multi-user.target
EOF

echo "==> Перезагрузка systemd..."
systemctl daemon-reload
systemctl enable ${SERVICE_NAME}

echo ""
echo "✅ Установка завершена"
echo ""
echo "👉 Теперь закинь файлы:"
echo "   ${BOT_DIR}/bot.py"
echo "   ${BOT_DIR}/.env"
echo ""
echo "👉 И запусти:"
echo "   systemctl restart ${SERVICE_NAME}"
echo "   journalctl -u ${SERVICE_NAME} -f"
