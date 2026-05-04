# Домашнее задание к занятию «GitLab» - Шевляков Алексей

### Задание 1
Что нужно сделать:

1. Разверните GitLab локально, используя Vagrantfile и инструкцию, описанные в этом репозитории.
2. Создайте новый проект и пустой репозиторий в нём.
3. Зарегистрируйте gitlab-runner для этого проекта и запустите его в режиме Docker. Раннер можно регистрировать и запускать на той же виртуальной машине, на которой запущен GitLab.
- В качестве ответа в репозиторий шаблона с решением добавьте скриншоты с настройками раннера в проекте.

##### Решение 1

* `Выполняем клонирование репозитория с шаблоном ДЗ к себе на ПК с помощью команды ***git clone***.`
```
git clone https://github.com/staintru/gitlab-hw/
```
* `Устанавливаем VirtualBox`
```
sudo apt install virtualbox
```
* `Находим и скачиваем пакет Vagrant`
[можно отсюда](https://hashicorp-releases.yandexcloud.net/vagrant/2.4.9/vagrant_2.4.9-1_amd64.deb)
* `Устанавливаем Vagrant`
```
sudo dpkg -i vagrant_2.4.9-1_amd64.deb
```
* `Редактируем Vagrantfile`
```
Vagrant.configure("2") do |config|
  ENV['VAGRANT_SERVER_URL'] = 'http://vagrant.elab.pro'
  config.vm.box = "ubuntu/jammy64"
  config.vm.network "private_network", ip: "192.168.56.10"
  config.vm.disk :disk, size: "15GB", primary: true
  config.vm.provider "virtualbox" do |vb|
    vb.memory = "6144"
  end
  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get install -y docker.io docker-compose
    apt-get install -y curl openssh-server ca-certificates tzdata perl
    curl -s https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash
    sudo EXTERNAL_URL="http://gitlab.localdomain" apt-get -y install gitlab-ce
    docker pull gitlab/gitlab-runner:latest
    docker pull sonarsource/sonar-scanner-cli:latest
    docker pull golang:1.17
    docker pull docker:latest
    sysctl vm.max_map_count=262144
    echo -e "192.168.56.10\tubuntu-bionic\tubuntu-bionic" | >> /etc/hosts
  SHELL
end
```
* `Запускаем vagrant дожидаемся окончания процесса`


```
VAGRANT_EXPERIMENTAL="disk" vagrant up
```
* `В браузере переходим на http://gitlab.localdomain или указываем адрес 192.168.56.10 и видим страницу входа GitLab`
![GitLab](https://github.com/staintru/gitlab-hw/auth.png)`
* `Находясь в директории, в которой запускали vagrant, вводим команду`
```
vagrant ssh
```
`попадаем на созданную ВМ. Там переходим в каталок /etc/gitlab и открываем файл initial_root_password, где храниться пароль Gitlab`
`На странице http://gitlab.localdomain вводим логин root и пароль указанный в файле`
`Попадаем "внутрь" видим отсутствие проектов`
![Пустой GitLab](https://github.com/staintru/gitlab-hw/GitLab.png)
* `Создаем новый пустой проект, вносим его в список удаленных репозиториев,`
```
git remote add my_git http://192.168.56.10/root/my_project.git
```
`заливаем в него клон`
```
git push my_git
```
* `Регистрируем раннер на созданной ВМ`
```
docker run -ti --rm --name gitlab-runner \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest register
```
![runner](https://github.com/staintru/gitlab-hw/runner_reg.png)
* `Вносим изменения в файле конфигурации srv/gitlab-runner/config.toml на созданной ВМ путем изменения строки`
```
volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock"]
```
![runner config](https://github.com/staintru/gitlab-hw/runner_conf.png)
* `Запускаем раннер на созданной ВМ`
```
docker run -d --name gitlab-runner --restart always \
     --network host \
     -v /srv/gitlab-runner/config:/etc/gitlab-runner \
     -v /var/run/docker.sock:/var/run/docker.sock \
     gitlab/gitlab-runner:latest
```
![runner run](https://github.com/staintru/gitlab-hw/runner_start.png)
* `Проверяем`
![runner](https://github.com/staintru/gitlab-hw/runner.png)
---

### Задание 2
`Запушьте репозиторий на GitLab, изменив origin. Это изучалось на занятии по Git.`
`Создайте .gitlab-ci.yml, описав в нём все необходимые, на ваш взгляд, этапы.`

##### Решение

* `Меняем origin`
```
git remote rename my_git my_git2
```
* `Создаем файл .gitlab-ci.yml с наполнением:`
```
stages:
  - test
  - build

test:
  stage: test
  image: golang:1.17
  script: 
   - go test .

build:
  stage: build
  image: docker:latest
  script:
   - docker build .
```
* `Пушим с новым origin`
```
git push my_git2
```
![my_git2](https://github.com/staintru/gitlab-hw/my_git.png)

* `Вносим правки`
![Pipeline_new](https://github.com/staintru/gitlab-hw/Pipeline_ch.png)
* `Повторная проверка`
![Pipeline2](https://github.com/staintru/gitlab-hw/Pipeline.png)


---
