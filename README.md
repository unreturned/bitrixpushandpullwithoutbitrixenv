# bitrixpushandpullwithoutbitrixenv
Push and Pull for Bitrix without BitrixEnv

## Как включить
Добавляем конфигурацию в секцию 
```
http {
    ...
}
```
например с помощью `include /path/to/push.conf`

## Сборка nginx с модулем nginx-push-stream-module
### Для Debian, Ubuntu
$$$ Для Debian по аналогии с http://www.lisov.org/dobavlenie-modulya-nginx-debian-way.html

### Для CentOS
Определим переменную (пользователя) builder, на будущее пригодится

`export MYUSER=builder`

Подключаем [официальные репозитории nginx](http://nginx.org/en/linux_packages.html#stable) (сразу вместе с необходимыми GPG-ключами)

`rpm -Uvh http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm`

Переключаем на исходный код

`sed -i 's|$basearch|SRPMS|g' /etc/yum.repos.d/nginx.repo`

Далее устанавливаем сборочные утилиты

`yum install rpmdevtools yum-utils gcc make`

Устанавливаем git, чтобы скачать исходники [nginx-push-stream-module](https://github.com/wandenberg/nginx-push-stream-module)

`yum install git`

Добавим пользователя, от которого будем собирать пакет (чтобы не нанести вред системе)

`adduser $MYUSER`
`sudo -u $MYUSER -i`

Готовим дерево сборки для пакетов

`rpmdev-setuptree`

Скачиваем SRPM и устанавливаем

`cd ~/rpmbuild/SRPMS/`
`yumdownloader --source nginx`

В моем случае скачался nginx-1.6.2-1.el6.ngx.src.rpm и установим его (он будет распакован в предварительно подготовленное дерево каталогов ~/rpmbuild)

`rpm -ivh nginx-1.6.2-1.el6.ngx.src.rpm`

Выходим из пользователя, скачиваем необходимые зависимости для сборки nginx и снова входим в пользователя (пользователь у нас myrpmbuilder)

```
exit
yum-builddep /home/$MYUSER/rpmbuild/SRPMS/nginx-1.6.2-1.el6.ngx.src.rpm --nogpgcheck
sudo -u $MYUSER -i
```

Переходим в директорию других для исходных кодов

`cd ~/rpmbuild/SOURCES/`

Клонируем репозиторий https://github.com/wandenberg/nginx-push-stream-module

`git clone https://github.com/wandenberg/nginx-push-stream-module`

Вносим изменения в сборку

`vim ~/rpmbuild/SPECS/nginx.spec`

Добавляем строчку

`Source1000: nginx-push-stream-module`

там где много Source.

И добавляем строчку

`--add-module=%{SOURCE1000} \`

в configure область (я добавил и в debug сразу).

И собираем пакет

```
cd ~/rpmbuild/SPECS/
rpmbuild -bb nginx.spec
```

Обязательно читаем вывод всего что вводим.

Готово! Пакет лежит по пути `~/rpmbuild/RPMS/x86_64/nginx-1.6.2-1.el6.ngx.x86_64.rpm`

Можно устанавливать от root, например командой

`rpm -Uvh /home/$MYUSER/rpmbuild/RPMS/x86_64/nginx-1.6.2-1.el6.ngx.x86_64.rpm`

## Полезные ссылки
- [https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=48&LESSON_ID=2033](https://dev.1c-bitrix.ru/learning/course/index.php?COURSE_ID=48&LESSON_ID=2033)
- [http://bitrix-guide.com/admin_modules/push_and_pull/](http://bitrix-guide.com/admin_modules/push_and_pull/)
- [http://dev.1c-bitrix.ru/community/blogs/product_features/why-configure-pushpull.php](http://dev.1c-bitrix.ru/community/blogs/product_features/why-configure-pushpull.php)
- [https://github.com/wandenberg/nginx-push-stream-module](https://github.com/wandenberg/nginx-push-stream-module)
