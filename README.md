# Установка ноды Subspace

1) Обновляем 'базу данных', обновляем дистрибутив
> sudo apt-get update && sudo apt-get upgrade -y

2) Скачиваем необходимые зависимости одной командой
 >sudo apt-get install wget jq ocl-icd-opencl-dev \
 libopencl-clang-dev libgomp1 ocl-icd-libopencl1 -y
 
3) Скачиваем исполняемые файлы и выводим их версии одной командой
>mkdir $HOME/subspace >/dev/null 2>&1 && \
cd $HOME/subspace && \
VER=$(wget -qO- https://api.github.com/repos/subspace/subspace-cli/releases | jq '.[] | select(.prerelease==false) | select(.draft==false) | .html_url' | grep -Eo "v[0-9]*.[0-9]*.[0-9]*-alpha" | head -n 1) && \
wget https://github.com/subspace/subspace-cli/releases/download/${VER}/subspace-cli-ubuntu-x86_64-${VER} -qO subspace; \
sudo chmod +x * && \
if [[ $(./subspace -h) == "" ]]; then
  echo -e "\n\ndat sh*t is broken, ping @cyberomanov.\n\n"
else
  sudo mv * /usr/local/bin/ && \
  echo -e "\n\nrelease >> ${VER}.\n\n"
fi && \
cd $HOME && \
rm -Rvf $HOME/subspace >/dev/null 2>&1

На выходе мы должны увидеть "release >> v0.1.7-alpha", если видим иное - повторяем пункт 2 до победного

4) Если у вас нет кошелька полкадота, то качаем(https://polkadot.js.org/extension/), если есть, то заходим на дашборд(https://polkadot.js.org/apps/?rpc=wss%3A%2F%2Feu-0.gemini-3c.subspace.network%2Fws#/accounts), 
нажимаем Метадата

![image](https://user-images.githubusercontent.com/59707245/225451107-2f888a7f-579c-484b-859a-298a2c5b80b0.png)

Обновить метадату

![image](https://user-images.githubusercontent.com/59707245/225451147-c9445258-97e0-489f-a401-e9b7c3fede27.png)

Подтверждаем добавление сети

![image](https://user-images.githubusercontent.com/59707245/225451223-ac6f737b-cb7e-494e-b9a1-830d269b3fd8.png)

Переключаемся на нашу сеть Сабспейса

![image](https://user-images.githubusercontent.com/59707245/225451273-24fbaf82-8d1c-40af-b9d7-0e539c3f7c44.png)

5) Инициализируемся
>subspace init

6) Вводим скопированный кошелек, желаемое имя пользователя, далее все настройки можно оставить по умолчанию

7)фиксим журнал
>sudo tee <<EOF >/dev/null /etc/systemd/journald.conf
Storage=persistent
EOF

8)рестартим журнал
>sudo systemctl restart systemd-journald

9) Cоздаём файл сервиса для запуска ноды
>sudo tee <<EOF >/dev/null /etc/systemd/system/subspaced.service
[Unit]
Description=Subspace Node
After=network.target
[Service]
Type=simple
User=$USER
ExecStart=$(which subspace) farm -v
Restart=always
RestartSec=10
LimitNOFILE=65535
[Install]
WantedBy=multi-user.target
EOF

10) Запускаем ноду
>sudo systemctl daemon-reload && \
sudo systemctl enable subspaced && \
sudo systemctl restart subspaced

11)Проверяем логи
>sudo journalctl -fu subspaced --no-hostname -o cat

За синхронизацией можно слежить при помощи команды
>sudo journalctl -fu subspaced -o cat | grep -E "best"

Ползеные команды:
Пезапуск фармера и ноды
>sudo systemctl restart subspaced

Остановка фармера и ноды
>sudo systemctl stop subspaced

Проверяем логи
>sudo journalctl -fu subspaced --no-hostname -o cat
