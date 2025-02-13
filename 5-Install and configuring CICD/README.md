## Установка и настройка CI/CD
```
Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:

Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
Автоматический деплой нового docker образа.
Можно использовать teamcity, jenkins, GitLab CI или GitHub Actions.

Ожидаемый результат:

Интерфейс ci/cd сервиса доступен по http.
При любом коммите в репозиторие с тестовым приложением происходит сборка и отправка в регистр Docker образа.
При создании тега (например, v1.0.0) происходит сборка и отправка с соответствующим label в регистри, а также деплой соответствующего Docker образа в кластер Kubernetes.
```
### Решение:

Настроим ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

### 5.1 Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.

Для автоматической сборки и docker-образа при коммите в репозиторий с тестовым приложением воспользуемся платформой CI/CD GitHub Actions.

Создадим в DockerHub секретный токен my token.

![img2](https://github.com/user-attachments/assets/0acd55f8-640a-409c-a69f-7c9ce840ccad)

Добавим токен в Settings -> Secrets and variables -> Actions secrets and variables в переменную MY_TOKEN_DOCKER_HUB.
В переменную USER_DOCKER_HUB добавим имя пользователя от Docker Hub slava1005.

![img3](https://github.com/user-attachments/assets/ea757faa-5f17-4d69-98f3-ab9fceec62ad)

Далее, создадим workflow файл для автоматической сборки приложения nginx:
Перейдем на вкладку Actions, выполним New workflow, затем Simple workflow (Config). Создадим файл ../.github/workflows/actions-build.yml.

actions-build.yml
```
name: Сборка Docker-образа

on:
  push:
    branches:
      - '*'
jobs:
  my_build_job:
    runs-on: ubuntu-latest

    steps:
      - name: Проверка кода
        uses: actions/checkout@v4

      - name: Вход на Docker Hub
        uses: docker/login-action@v2
        with:
          username: ${{ secrets.USER_DOCKER_HUB }}
          password: ${{ secrets.MY_TOKEN_DOCKER_HUB }}

      - name: Сборка Docker-образа
        run: |
          docker build . --file Dockerfile --tag nginx:latest
          docker tag nginx:latest ${{ secrets.USER_DOCKER_HUB }}/nginx:latest

      - name: Push Docker-образа в Docker Hub
        run: |
          docker push ${{ secrets.USER_DOCKER_HUB }}/nginx:latest
```
Выполним commit.

Перейдем в раздел Actions, где видим, что Workflow file успешно выполнился.
![img4](https://github.com/user-attachments/assets/51206075-fea0-45a7-ba89-d2e9af8726c2)

<details>
<summary>Лог выполнения Workflow file:</summary>
  
```
2025-02-13T05:44:32.4342069Z Current runner version: '2.322.0'
2025-02-13T05:44:32.4365850Z ##[group]Operating System
2025-02-13T05:44:32.4366595Z Ubuntu
2025-02-13T05:44:32.4367371Z 24.04.1
2025-02-13T05:44:32.4367895Z LTS
2025-02-13T05:44:32.4368424Z ##[endgroup]
2025-02-13T05:44:32.4368989Z ##[group]Runner Image
2025-02-13T05:44:32.4369570Z Image: ubuntu-24.04
2025-02-13T05:44:32.4370158Z Version: 20250209.1.0
2025-02-13T05:44:32.4371211Z Included Software: https://github.com/actions/runner-images/blob/ubuntu24/20250209.1/images/ubuntu/Ubuntu2404-Readme.md
2025-02-13T05:44:32.4372591Z Image Release: https://github.com/actions/runner-images/releases/tag/ubuntu24%2F20250209.1
2025-02-13T05:44:32.4373599Z ##[endgroup]
2025-02-13T05:44:32.4374141Z ##[group]Runner Image Provisioner
2025-02-13T05:44:32.4374785Z 2.0.422.1
2025-02-13T05:44:32.4375313Z ##[endgroup]
2025-02-13T05:44:32.4376385Z ##[group]GITHUB_TOKEN Permissions
2025-02-13T05:44:32.4378343Z Contents: read
2025-02-13T05:44:32.4378924Z Metadata: read
2025-02-13T05:44:32.4379739Z Packages: read
2025-02-13T05:44:32.4380337Z ##[endgroup]
2025-02-13T05:44:32.4383205Z Secret source: Actions
2025-02-13T05:44:32.4383909Z Prepare workflow directory
2025-02-13T05:44:32.4682993Z Prepare all required actions
2025-02-13T05:44:32.4718603Z Getting action download info
2025-02-13T05:44:32.6845923Z ##[group]Download immutable action package 'actions/checkout@v4'
2025-02-13T05:44:32.6847078Z Version: 4.2.2
2025-02-13T05:44:32.6848128Z Digest: sha256:ccb2698953eaebd21c7bf6268a94f9c26518a7e38e27e0b83c1fe1ad049819b1
2025-02-13T05:44:32.6849456Z Source commit SHA: 11bd71901bbe5b1630ceea73d27597364c9af683
2025-02-13T05:44:32.6850222Z ##[endgroup]
2025-02-13T05:44:32.7706596Z ##[group]Download immutable action package 'docker/login-action@v2'
2025-02-13T05:44:32.7707791Z Version: 2.2.0
2025-02-13T05:44:32.7708573Z Digest: sha256:074da69a9e77797ae469f40db0f64632c02629b61bfa229f2fa803f41a464283
2025-02-13T05:44:32.7709614Z Source commit SHA: 465a07811f14bebb1938fbed4728c6a1ff8901fc
2025-02-13T05:44:32.7710392Z ##[endgroup]
2025-02-13T05:44:33.0817569Z Complete job name: my_build_job
2025-02-13T05:44:33.1507984Z ##[group]Run actions/checkout@v4
2025-02-13T05:44:33.1508829Z with:
2025-02-13T05:44:33.1509329Z   repository: ***/Test-application
2025-02-13T05:44:33.1509962Z   token: ***
2025-02-13T05:44:33.1510380Z   ssh-strict: true
2025-02-13T05:44:33.1510797Z   ssh-user: git
2025-02-13T05:44:33.1511233Z   persist-credentials: true
2025-02-13T05:44:33.1511709Z   clean: true
2025-02-13T05:44:33.1512133Z   sparse-checkout-cone-mode: true
2025-02-13T05:44:33.1512636Z   fetch-depth: 1
2025-02-13T05:44:33.1513048Z   fetch-tags: false
2025-02-13T05:44:33.1513469Z   show-progress: true
2025-02-13T05:44:33.1513911Z   lfs: false
2025-02-13T05:44:33.1514310Z   submodules: false
2025-02-13T05:44:33.1514743Z   set-safe-directory: true
2025-02-13T05:44:33.1515405Z ##[endgroup]
2025-02-13T05:44:33.3636528Z Syncing repository: ***/Test-application
2025-02-13T05:44:33.3638721Z ##[group]Getting Git version info
2025-02-13T05:44:33.3639685Z Working directory is '/home/runner/work/Test-application/Test-application'
2025-02-13T05:44:33.3640861Z [command]/usr/bin/git version
2025-02-13T05:44:33.3688306Z git version 2.48.1
2025-02-13T05:44:33.3716469Z ##[endgroup]
2025-02-13T05:44:33.3732665Z Temporarily overriding HOME='/home/runner/work/_temp/f9f991aa-982d-43c2-bd0c-6278e0df6328' before making global git config changes
2025-02-13T05:44:33.3735502Z Adding repository directory to the temporary git global config as a safe directory
2025-02-13T05:44:33.3746348Z [command]/usr/bin/git config --global --add safe.directory /home/runner/work/Test-application/Test-application
2025-02-13T05:44:33.3781067Z Deleting the contents of '/home/runner/work/Test-application/Test-application'
2025-02-13T05:44:33.3784702Z ##[group]Initializing the repository
2025-02-13T05:44:33.3790085Z [command]/usr/bin/git init /home/runner/work/Test-application/Test-application
2025-02-13T05:44:33.3873924Z hint: Using 'master' as the name for the initial branch. This default branch name
2025-02-13T05:44:33.3875555Z hint: is subject to change. To configure the initial branch name to use in all
2025-02-13T05:44:33.3876517Z hint: of your new repositories, which will suppress this warning, call:
2025-02-13T05:44:33.3877557Z hint:
2025-02-13T05:44:33.3878095Z hint: 	git config --global init.defaultBranch <name>
2025-02-13T05:44:33.3879038Z hint:
2025-02-13T05:44:33.3880037Z hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
2025-02-13T05:44:33.3881444Z hint: 'development'. The just-created branch can be renamed via this command:
2025-02-13T05:44:33.3883011Z hint:
2025-02-13T05:44:33.3883898Z hint: 	git branch -m <name>
2025-02-13T05:44:33.3885645Z Initialized empty Git repository in /home/runner/work/Test-application/Test-application/.git/
2025-02-13T05:44:33.3893718Z [command]/usr/bin/git remote add origin https://github.com/***/Test-application
2025-02-13T05:44:33.3931534Z ##[endgroup]
2025-02-13T05:44:33.3932934Z ##[group]Disabling automatic garbage collection
2025-02-13T05:44:33.3934658Z [command]/usr/bin/git config --local gc.auto 0
2025-02-13T05:44:33.3964260Z ##[endgroup]
2025-02-13T05:44:33.3965492Z ##[group]Setting up auth
2025-02-13T05:44:33.3970786Z [command]/usr/bin/git config --local --name-only --get-regexp core\.sshCommand
2025-02-13T05:44:33.3999888Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'core\.sshCommand' && git config --local --unset-all 'core.sshCommand' || :"
2025-02-13T05:44:33.4303504Z [command]/usr/bin/git config --local --name-only --get-regexp http\.https\:\/\/github\.com\/\.extraheader
2025-02-13T05:44:33.4332994Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'http\.https\:\/\/github\.com\/\.extraheader' && git config --local --unset-all 'http.https://github.com/.extraheader' || :"
2025-02-13T05:44:33.4561070Z [command]/usr/bin/git config --local http.https://github.com/.extraheader AUTHORIZATION: basic ***
2025-02-13T05:44:33.4595895Z ##[endgroup]
2025-02-13T05:44:33.4604626Z ##[group]Fetching the repository
2025-02-13T05:44:33.4606105Z [command]/usr/bin/git -c protocol.version=2 fetch --no-tags --prune --no-recurse-submodules --depth=1 origin +c6bc9192477235443d6e4f6087a0037fb3636b5f:refs/remotes/origin/main
2025-02-13T05:44:33.9622380Z From https://github.com/***/Test-application
2025-02-13T05:44:33.9623838Z  * [new ref]         c6bc9192477235443d6e4f6087a0037fb3636b5f -> origin/main
2025-02-13T05:44:33.9665446Z ##[endgroup]
2025-02-13T05:44:33.9666766Z ##[group]Determining the checkout info
2025-02-13T05:44:33.9668796Z ##[endgroup]
2025-02-13T05:44:33.9676592Z [command]/usr/bin/git sparse-checkout disable
2025-02-13T05:44:33.9717778Z [command]/usr/bin/git config --local --unset-all extensions.worktreeConfig
2025-02-13T05:44:33.9746595Z ##[group]Checking out the ref
2025-02-13T05:44:33.9751082Z [command]/usr/bin/git checkout --progress --force -B main refs/remotes/origin/main
2025-02-13T05:44:33.9794313Z Switched to a new branch 'main'
2025-02-13T05:44:33.9798322Z branch 'main' set up to track 'origin/main'.
2025-02-13T05:44:33.9804667Z ##[endgroup]
2025-02-13T05:44:33.9838702Z [command]/usr/bin/git log -1 --format=%H
2025-02-13T05:44:33.9861347Z c6bc9192477235443d6e4f6087a0037fb3636b5f
2025-02-13T05:44:34.0072702Z ##[group]Run docker/login-action@v2
2025-02-13T05:44:34.0073320Z with:
2025-02-13T05:44:34.0073832Z   username: ***
2025-02-13T05:44:34.0074363Z   password: ***
2025-02-13T05:44:34.0074792Z   ecr: auto
2025-02-13T05:44:34.0075205Z   logout: true
2025-02-13T05:44:34.0075643Z ##[endgroup]
2025-02-13T05:44:34.1010008Z Logging into Docker Hub...
2025-02-13T05:44:34.9079979Z Login Succeeded!
2025-02-13T05:44:34.9298248Z ##[group]Run docker build . --file Dockerfile --tag nginx:latest
2025-02-13T05:44:34.9300296Z [36;1mdocker build . --file Dockerfile --tag nginx:latest[0m
2025-02-13T05:44:34.9302577Z [36;1mdocker tag nginx:latest ***/nginx:latest[0m
2025-02-13T05:44:34.9455988Z shell: /usr/bin/bash -e {0}
2025-02-13T05:44:34.9457769Z ##[endgroup]
2025-02-13T05:44:35.4327392Z #0 building with "default" instance using docker driver
2025-02-13T05:44:35.4328892Z 
2025-02-13T05:44:35.4329674Z #1 [internal] load build definition from Dockerfile
2025-02-13T05:44:35.4331098Z #1 transferring dockerfile: 99B done
2025-02-13T05:44:35.4332298Z #1 DONE 0.0s
2025-02-13T05:44:35.4332814Z 
2025-02-13T05:44:35.4333576Z #2 [internal] load metadata for docker.io/library/nginx:1.27-alpine
2025-02-13T05:44:35.6603200Z #2 ...
2025-02-13T05:44:35.6603985Z 
2025-02-13T05:44:35.6604846Z #3 [auth] library/nginx:pull token for registry-1.docker.io
2025-02-13T05:44:35.6606530Z #3 DONE 0.0s
2025-02-13T05:44:35.8105849Z 
2025-02-13T05:44:35.8107150Z #2 [internal] load metadata for docker.io/library/nginx:1.27-alpine
2025-02-13T05:44:36.5463057Z #2 DONE 1.3s
2025-02-13T05:44:36.6634843Z 
2025-02-13T05:44:36.6635455Z #4 [internal] load .dockerignore
2025-02-13T05:44:36.6635985Z #4 transferring context: 2B done
2025-02-13T05:44:36.6636419Z #4 DONE 0.0s
2025-02-13T05:44:36.6636665Z 
2025-02-13T05:44:36.6637031Z #5 [internal] load build context
2025-02-13T05:44:36.6637336Z #5 transferring context: 135B done
2025-02-13T05:44:36.6637591Z #5 DONE 0.0s
2025-02-13T05:44:36.6637707Z 
2025-02-13T05:44:36.6638118Z #6 [1/2] FROM docker.io/library/nginx:1.27-alpine@sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef
2025-02-13T05:44:36.6638952Z #6 resolve docker.io/library/nginx:1.27-alpine@sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef done
2025-02-13T05:44:36.6639733Z #6 sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef 10.36kB / 10.36kB done
2025-02-13T05:44:36.6640426Z #6 sha256:d41a14a4ecff96bdae6253ad2f58d8f258786db438307846081e8d835b984111 11.23kB / 11.23kB done
2025-02-13T05:44:36.6641094Z #6 sha256:6666d93f054a3f4315894b76f2023f3da2fcb5ceb5f8d91625cca81623edd2da 2.50kB / 2.50kB done
2025-02-13T05:44:36.6641760Z #6 sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2ded61476c24403d592134f0 0B / 3.64MB 0.1s
2025-02-13T05:44:36.6642417Z #6 sha256:5215a08fb124932f41461b5da64ff183d8772c8cfa6d2ba64447f60275113f1b 0B / 1.79MB 0.1s
2025-02-13T05:44:36.6643033Z #6 sha256:f8813b38090d755304b9b62b790b0f19cd7eee409c3d0a3d495411be8eb35408 0B / 627B 0.1s
2025-02-13T05:44:36.8462080Z #6 extracting sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2ded61476c24403d592134f0
2025-02-13T05:44:36.9592104Z #6 sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2ded61476c24403d592134f0 3.64MB / 3.64MB 0.3s done
2025-02-13T05:44:36.9594620Z #6 sha256:5215a08fb124932f41461b5da64ff183d8772c8cfa6d2ba64447f60275113f1b 1.79MB / 1.79MB 0.2s done
2025-02-13T05:44:36.9596074Z #6 sha256:f8813b38090d755304b9b62b790b0f19cd7eee409c3d0a3d495411be8eb35408 627B / 627B 0.3s done
2025-02-13T05:44:36.9597412Z #6 extracting sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2ded61476c24403d592134f0 0.1s done
2025-02-13T05:44:36.9598491Z #6 sha256:e92b9802c411990b7ff95ce238004213125726f1586227fe2665b765cd833602 0B / 404B 0.3s
2025-02-13T05:44:36.9599545Z #6 sha256:9f41882e104d5e4b0e443fc387e440da00594e400a8c6c8d426e86a910e11084 0B / 956B 0.3s
2025-02-13T05:44:36.9600643Z #6 sha256:4b56e0e1b50da94d716fc880809f96d5cb2751e0d9d9c680ae6c60e0f9239708 0B / 1.21kB 0.3s
2025-02-13T05:44:36.9601698Z #6 extracting sha256:5215a08fb124932f41461b5da64ff183d8772c8cfa6d2ba64447f60275113f1b
2025-02-13T05:44:37.0626318Z #6 sha256:e92b9802c411990b7ff95ce238004213125726f1586227fe2665b765cd833602 404B / 404B 0.5s done
2025-02-13T05:44:37.0627657Z #6 sha256:9f41882e104d5e4b0e443fc387e440da00594e400a8c6c8d426e86a910e11084 956B / 956B 0.4s done
2025-02-13T05:44:37.0629360Z #6 extracting sha256:5215a08fb124932f41461b5da64ff183d8772c8cfa6d2ba64447f60275113f1b 0.0s done
2025-02-13T05:44:37.0630400Z #6 sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 0B / 15.37MB 0.5s
2025-02-13T05:44:37.0631344Z #6 sha256:5281c445f8b7ca564e8124519f166a63c67a38074755a2d72738be7800dbbc78 0B / 1.40kB 0.5s
2025-02-13T05:44:37.1633474Z #6 sha256:4b56e0e1b50da94d716fc880809f96d5cb2751e0d9d9c680ae6c60e0f9239708 1.21kB / 1.21kB 0.5s done
2025-02-13T05:44:37.1635036Z #6 extracting sha256:f8813b38090d755304b9b62b790b0f19cd7eee409c3d0a3d495411be8eb35408 done
2025-02-13T05:44:37.1636022Z #6 extracting sha256:9f41882e104d5e4b0e443fc387e440da00594e400a8c6c8d426e86a910e11084 done
2025-02-13T05:44:37.3627723Z #6 sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 5.24MB / 15.37MB 0.8s
2025-02-13T05:44:37.3628570Z #6 sha256:5281c445f8b7ca564e8124519f166a63c67a38074755a2d72738be7800dbbc78 1.40kB / 1.40kB 0.7s done
2025-02-13T05:44:37.3629341Z #6 extracting sha256:e92b9802c411990b7ff95ce238004213125726f1586227fe2665b765cd833602 done
2025-02-13T05:44:37.3630076Z #6 extracting sha256:4b56e0e1b50da94d716fc880809f96d5cb2751e0d9d9c680ae6c60e0f9239708 done
2025-02-13T05:44:37.3630762Z #6 extracting sha256:5281c445f8b7ca564e8124519f166a63c67a38074755a2d72738be7800dbbc78 done
2025-02-13T05:44:37.5579930Z #6 sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 9.44MB / 15.37MB 0.9s
2025-02-13T05:44:37.5581185Z #6 extracting sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413
2025-02-13T05:44:37.6591216Z #6 sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 15.37MB / 15.37MB 1.0s done
2025-02-13T05:44:37.9698503Z #6 extracting sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 0.3s done
2025-02-13T05:44:37.9699055Z #6 DONE 1.4s
2025-02-13T05:44:38.0732650Z 
2025-02-13T05:44:38.0733551Z #7 [2/2] COPY index.html /usr/share/nginx/html
2025-02-13T05:44:38.0734319Z #7 DONE 0.0s
2025-02-13T05:44:38.0734790Z 
2025-02-13T05:44:38.0735184Z #8 exporting to image
2025-02-13T05:44:38.0735818Z #8 exporting layers 0.1s done
2025-02-13T05:44:38.0982307Z #8 writing image sha256:7c7e6d4935d0eed4e40bce1b29b1b76fa4fbdab3741c9e2b6d425445ecef782a done
2025-02-13T05:44:38.0983219Z #8 naming to docker.io/library/nginx:latest done
2025-02-13T05:44:38.0983783Z #8 DONE 0.1s
2025-02-13T05:44:38.1221423Z ##[group]Run docker push ***/nginx:latest
2025-02-13T05:44:38.1221925Z [36;1mdocker push ***/nginx:latest[0m
2025-02-13T05:44:38.1280063Z shell: /usr/bin/bash -e {0}
2025-02-13T05:44:38.1280501Z ##[endgroup]
2025-02-13T05:44:38.1609527Z The push refers to repository [docker.io/***/nginx]
2025-02-13T05:44:38.3834780Z a4f7eda351a9: Preparing
2025-02-13T05:44:38.3837617Z 72120687062c: Preparing
2025-02-13T05:44:38.3838623Z 469fc702bc62: Preparing
2025-02-13T05:44:38.3839378Z 74964efcae21: Preparing
2025-02-13T05:44:38.3840118Z ad4f5bc987ca: Preparing
2025-02-13T05:44:38.3840843Z ef050c9a03b5: Preparing
2025-02-13T05:44:38.3841360Z 83c20bc61eb8: Preparing
2025-02-13T05:44:38.3841849Z 1024e8977b69: Preparing
2025-02-13T05:44:38.3842416Z a0904247e36a: Preparing
2025-02-13T05:44:38.3842900Z ef050c9a03b5: Waiting
2025-02-13T05:44:38.3843387Z 83c20bc61eb8: Waiting
2025-02-13T05:44:38.3843944Z 1024e8977b69: Waiting
2025-02-13T05:44:38.3844433Z a0904247e36a: Waiting
2025-02-13T05:44:38.9639083Z 469fc702bc62: Layer already exists
2025-02-13T05:44:38.9738961Z 74964efcae21: Layer already exists
2025-02-13T05:44:38.9917843Z 72120687062c: Layer already exists
2025-02-13T05:44:38.9940023Z ad4f5bc987ca: Layer already exists
2025-02-13T05:44:39.3341529Z 83c20bc61eb8: Layer already exists
2025-02-13T05:44:39.3445417Z ef050c9a03b5: Layer already exists
2025-02-13T05:44:39.3653261Z a0904247e36a: Layer already exists
2025-02-13T05:44:39.3823788Z 1024e8977b69: Layer already exists
2025-02-13T05:44:40.6402674Z a4f7eda351a9: Pushed
2025-02-13T05:44:43.8778850Z latest: digest: sha256:54092ccfd53f516984da80cb54a2ae5906f0226829e9f7aa9dca0bffc0f8d99e size: 2196
2025-02-13T05:44:43.8862318Z Post job cleanup.
2025-02-13T05:44:43.9825501Z [command]/usr/bin/docker logout 
2025-02-13T05:44:43.9942907Z Removing login credentials for https://index.docker.io/v1/
2025-02-13T05:44:44.0085448Z Post job cleanup.
2025-02-13T05:44:44.1008786Z [command]/usr/bin/git version
2025-02-13T05:44:44.1046596Z git version 2.48.1
2025-02-13T05:44:44.1090589Z Temporarily overriding HOME='/home/runner/work/_temp/af97b39c-e799-4c2e-adb9-5bd0f32ecc31' before making global git config changes
2025-02-13T05:44:44.1092235Z Adding repository directory to the temporary git global config as a safe directory
2025-02-13T05:44:44.1097423Z [command]/usr/bin/git config --global --add safe.directory /home/runner/work/Test-application/Test-application
2025-02-13T05:44:44.1132834Z [command]/usr/bin/git config --local --name-only --get-regexp core\.sshCommand
2025-02-13T05:44:44.1165094Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'core\.sshCommand' && git config --local --unset-all 'core.sshCommand' || :"
2025-02-13T05:44:44.1395984Z [command]/usr/bin/git config --local --name-only --get-regexp http\.https\:\/\/github\.com\/\.extraheader
2025-02-13T05:44:44.1416295Z http.https://github.com/.extraheader
2025-02-13T05:44:44.1428824Z [command]/usr/bin/git config --local --unset-all http.https://github.com/.extraheader
2025-02-13T05:44:44.1458697Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'http\.https\:\/\/github\.com\/\.extraheader' && git config --local --unset-all 'http.https://github.com/.extraheader' || :"
2025-02-13T05:44:44.1787550Z Cleaning up orphan processes
```
 </details>
 
В свою очередь на docker hub появляется загруженный образ:
![img5](https://github.com/user-attachments/assets/e64e3e19-26fc-4502-8e98-d1c7e9dfda48)

### 5.2 Автоматический деплой нового docker образа.

 В переменную KUBECONFIG добавим данные с сертификатами для доступа к кластеру из конфигурационного файла ~/.kube/config изменив при это серый ip-адрес master-node на белый.

Cоздадим workflow файл для автоматической сборки приложения nginx при создании тега, а также его автоматического развертывания в кластер Kubernetes.

actions-build-deployment.yml
```
name: Сборка и Развертывание
on:
  push:
    branches:
      - '*'
  create:
    tags:
      - '*'

env:
  IMAGE_TAG: slava1005/nginx

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Проверка кода
        uses: actions/checkout@v4

      - name: Установка Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Вход на Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.USER_DOCKER_HUB }}
          password: ${{ secrets.MY_TOKEN_DOCKER_HUB }}

      - name: Определяем версию
        run: |
          echo "GITHUB_REF: ${GITHUB_REF}"
          if [[ "${GITHUB_REF}" == refs/tags/* ]]; then
            VERSION=${GITHUB_REF#refs/tags/}
          else
            VERSION=$(git log -1 --pretty=format:%B | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+' || echo "")
          fi
          if [[ -z "$VERSION" ]]; then
            echo "No version found in the commit message or tag"
            exit 1
          fi
          VERSION=${VERSION//[[:space:]]/}  # Remove any spaces
          echo "Using version: $VERSION"
          echo "VERSION=${VERSION}" >> $GITHUB_ENV

      - name: Сборка и push
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: ${{ env.IMAGE_TAG }}:${{ env.VERSION }}

  deploy:
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Проверка кода
        uses: actions/checkout@v4

      - name: Установка kubectl
        run: |
          curl -LO "https://dl.k8s.io/release/v1.30.3/bin/linux/amd64/kubectl"
          chmod +x ./kubectl
          sudo mv ./kubectl /usr/local/bin/kubectl
          kubectl version --client

      - name: Конфигурирование kubectl, развертыввание и деплой
        run: |
          echo "${{ secrets.KUBECONFIG }}" > config.yml
          export KUBECONFIG=config.yml
          kubectl config view
          kubectl get nodes
          kubectl get pods --all-namespaces
          kubectl create deployment nginx --image=baryshnikovnv/nginx:1.0.0
          kubectl rollout status deployment.v1.apps/nginx
    env:
      KUBECONFIG: ${{ secrets.KUBECONFIG }}
```

Выполним commit c лэйблом v1.0.3.

![img7](https://github.com/user-attachments/assets/c0845423-c942-43d6-80ee-d6ff9d3d4b2e)
<details>
<summary>Лог выполнения Workflow file jobs build:</summary>
```
2025-02-12T05:44:30.3979706Z Current runner version: '2.322.0'
2025-02-12T05:44:30.4005205Z ##[group]Operating System
2025-02-12T05:44:30.4005989Z Ubuntu
2025-02-12T05:44:30.4006495Z 24.04.1
2025-02-12T05:44:30.4007126Z LTS
2025-02-12T05:44:30.4007629Z ##[endgroup]
2025-02-12T05:44:30.4008297Z ##[group]Runner Image
2025-02-12T05:44:30.4008941Z Image: ubuntu-24.04
2025-02-12T05:44:30.4009518Z Version: 20250209.1.0
2025-02-12T05:44:30.4010644Z Included Software: https://github.com/actions/runner-images/blob/ubuntu24/20250209.1/images/ubuntu/Ubuntu2404-Readme.md
2025-02-12T05:44:30.4012211Z Image Release: https://github.com/actions/runner-images/releases/tag/ubuntu24%2F20250209.1
2025-02-12T05:44:30.4013163Z ##[endgroup]
2025-02-12T05:44:30.4013774Z ##[group]Runner Image Provisioner
2025-02-12T05:44:30.4014459Z 2.0.422.1
2025-02-12T05:44:30.4014971Z ##[endgroup]
2025-02-12T05:44:30.4016191Z ##[group]GITHUB_TOKEN Permissions
2025-02-12T05:44:30.4017985Z Contents: read
2025-02-12T05:44:30.4018701Z Metadata: read
2025-02-12T05:44:30.4019454Z Packages: read
2025-02-12T05:44:30.4020241Z ##[endgroup]
2025-02-12T05:44:30.4023623Z Secret source: Actions
2025-02-12T05:44:30.4024831Z Prepare workflow directory
2025-02-12T05:44:30.4390364Z Prepare all required actions
2025-02-12T05:44:30.4426764Z Getting action download info
2025-02-12T05:44:30.6598963Z ##[group]Download immutable action package 'actions/checkout@v4'
2025-02-12T05:44:30.6600173Z Version: 4.2.2
2025-02-12T05:44:30.6601254Z Digest: sha256:ccb2698953eaebd21c7bf6268a94f9c26518a7e38e27e0b83c1fe1ad049819b1
2025-02-12T05:44:30.6602744Z Source commit SHA: 11bd71901bbe5b1630ceea73d27597364c9af683
2025-02-12T05:44:30.6603643Z ##[endgroup]
2025-02-12T05:44:30.7376412Z ##[group]Download immutable action package 'docker/setup-buildx-action@v3'
2025-02-12T05:44:30.7377370Z Version: 3.9.0
2025-02-12T05:44:30.7378333Z Digest: sha256:e49ac031513ae77f4ead0268d8db3db1ba21b1678dc2e6d83fda00118e7e141a
2025-02-12T05:44:30.7379440Z Source commit SHA: f7ce87c1d6bead3e36075b2ce75da1f6cc28aaca
2025-02-12T05:44:30.7380248Z ##[endgroup]
2025-02-12T05:44:31.0155077Z ##[group]Download immutable action package 'docker/login-action@v3'
2025-02-12T05:44:31.0155877Z Version: 3.3.0
2025-02-12T05:44:31.0156818Z Digest: sha256:d3b50b155c2bf53662cb43dca9f2f346ee3dd1283b16a526404daf25779f3d15
2025-02-12T05:44:31.0157813Z Source commit SHA: 9780b0c442fbb1117ed29e0efdff1e18412f7567
2025-02-12T05:44:31.0158533Z ##[endgroup]
2025-02-12T05:44:31.2901738Z ##[group]Download immutable action package 'docker/build-push-action@v5'
2025-02-12T05:44:31.2902864Z Version: 5.4.0
2025-02-12T05:44:31.2903656Z Digest: sha256:b1eb22c8ac697d9d0cdfecd96982afc33f5bf7897b1a76f89fd6cb58e9f98e3e
2025-02-12T05:44:31.2904682Z Source commit SHA: ca052bb54ab0790a636c9b5f226502c73d547a25
2025-02-12T05:44:31.2905449Z ##[endgroup]
2025-02-12T05:44:31.6241516Z Complete job name: build
2025-02-12T05:44:31.7008500Z ##[group]Run actions/checkout@v4
2025-02-12T05:44:31.7009671Z with:
2025-02-12T05:44:31.7010550Z   repository: ***/Test-application
2025-02-12T05:44:31.7011549Z   token: ***
2025-02-12T05:44:31.7012441Z   ssh-strict: true
2025-02-12T05:44:31.7013225Z   ssh-user: git
2025-02-12T05:44:31.7013883Z   persist-credentials: true
2025-02-12T05:44:31.7014671Z   clean: true
2025-02-12T05:44:31.7015364Z   sparse-checkout-cone-mode: true
2025-02-12T05:44:31.7016148Z   fetch-depth: 1
2025-02-12T05:44:31.7016844Z   fetch-tags: false
2025-02-12T05:44:31.7017524Z   show-progress: true
2025-02-12T05:44:31.7018205Z   lfs: false
2025-02-12T05:44:31.7018862Z   submodules: false
2025-02-12T05:44:31.7019516Z   set-safe-directory: true
2025-02-12T05:44:31.7020559Z env:
2025-02-12T05:44:31.7021370Z   IMAGE_TAG: ***/nginx
2025-02-12T05:44:31.7022213Z ##[endgroup]
2025-02-12T05:44:31.9057179Z Syncing repository: ***/Test-application
2025-02-12T05:44:31.9059822Z ##[group]Getting Git version info
2025-02-12T05:44:31.9061254Z Working directory is '/home/runner/work/Test-application/Test-application'
2025-02-12T05:44:31.9063657Z [command]/usr/bin/git version
2025-02-12T05:44:31.9121757Z git version 2.48.1
2025-02-12T05:44:31.9153070Z ##[endgroup]
2025-02-12T05:44:31.9169497Z Temporarily overriding HOME='/home/runner/work/_temp/133c315d-10b6-4f47-ae17-f998617d8b0c' before making global git config changes
2025-02-12T05:44:31.9172728Z Adding repository directory to the temporary git global config as a safe directory
2025-02-12T05:44:31.9176443Z [command]/usr/bin/git config --global --add safe.directory /home/runner/work/Test-application/Test-application
2025-02-12T05:44:31.9223790Z Deleting the contents of '/home/runner/work/Test-application/Test-application'
2025-02-12T05:44:31.9227680Z ##[group]Initializing the repository
2025-02-12T05:44:31.9231763Z [command]/usr/bin/git init /home/runner/work/Test-application/Test-application
2025-02-12T05:44:31.9309107Z hint: Using 'master' as the name for the initial branch. This default branch name
2025-02-12T05:44:31.9312873Z hint: is subject to change. To configure the initial branch name to use in all
2025-02-12T05:44:31.9316154Z hint: of your new repositories, which will suppress this warning, call:
2025-02-12T05:44:31.9318020Z hint:
2025-02-12T05:44:31.9319462Z hint: 	git config --global init.defaultBranch <name>
2025-02-12T05:44:31.9321420Z hint:
2025-02-12T05:44:31.9323198Z hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
2025-02-12T05:44:31.9326084Z hint: 'development'. The just-created branch can be renamed via this command:
2025-02-12T05:44:31.9328050Z hint:
2025-02-12T05:44:31.9329239Z hint: 	git branch -m <name>
2025-02-12T05:44:31.9331437Z Initialized empty Git repository in /home/runner/work/Test-application/Test-application/.git/
2025-02-12T05:44:31.9336565Z [command]/usr/bin/git remote add origin https://github.com/***/Test-application
2025-02-12T05:44:31.9363292Z ##[endgroup]
2025-02-12T05:44:31.9365733Z ##[group]Disabling automatic garbage collection
2025-02-12T05:44:31.9367690Z [command]/usr/bin/git config --local gc.auto 0
2025-02-12T05:44:31.9397480Z ##[endgroup]
2025-02-12T05:44:31.9399326Z ##[group]Setting up auth
2025-02-12T05:44:31.9402255Z [command]/usr/bin/git config --local --name-only --get-regexp core\.sshCommand
2025-02-12T05:44:31.9432947Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'core\.sshCommand' && git config --local --unset-all 'core.sshCommand' || :"
2025-02-12T05:44:31.9723268Z [command]/usr/bin/git config --local --name-only --get-regexp http\.https\:\/\/github\.com\/\.extraheader
2025-02-12T05:44:31.9757456Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'http\.https\:\/\/github\.com\/\.extraheader' && git config --local --unset-all 'http.https://github.com/.extraheader' || :"
2025-02-12T05:44:31.9986955Z [command]/usr/bin/git config --local http.https://github.com/.extraheader AUTHORIZATION: basic ***
2025-02-12T05:44:32.0025802Z ##[endgroup]
2025-02-12T05:44:32.0028843Z ##[group]Fetching the repository
2025-02-12T05:44:32.0037764Z [command]/usr/bin/git -c protocol.version=2 fetch --no-tags --prune --no-recurse-submodules --depth=1 origin +c6bc9192477235443d6e4f6087a0037fb3636b5f:refs/remotes/origin/main
2025-02-12T05:44:32.2794131Z From https://github.com/***/Test-application
2025-02-12T05:44:32.2796495Z  * [new ref]         c6bc9192477235443d6e4f6087a0037fb3636b5f -> origin/main
2025-02-12T05:44:32.2819540Z ##[endgroup]
2025-02-12T05:44:32.2822504Z ##[group]Determining the checkout info
2025-02-12T05:44:32.2825263Z ##[endgroup]
2025-02-12T05:44:32.2827063Z [command]/usr/bin/git sparse-checkout disable
2025-02-12T05:44:32.2870227Z [command]/usr/bin/git config --local --unset-all extensions.worktreeConfig
2025-02-12T05:44:32.2901278Z ##[group]Checking out the ref
2025-02-12T05:44:32.2905085Z [command]/usr/bin/git checkout --progress --force -B main refs/remotes/origin/main
2025-02-12T05:44:32.2951269Z Switched to a new branch 'main'
2025-02-12T05:44:32.2954795Z branch 'main' set up to track 'origin/main'.
2025-02-12T05:44:32.2962484Z ##[endgroup]
2025-02-12T05:44:32.2996773Z [command]/usr/bin/git log -1 --format=%H
2025-02-12T05:44:32.3019540Z c6bc9192477235443d6e4f6087a0037fb3636b5f
2025-02-12T05:44:32.3282675Z ##[group]Run docker/setup-buildx-action@v3
2025-02-12T05:44:32.3283850Z with:
2025-02-12T05:44:32.3284629Z   driver: docker-container
2025-02-12T05:44:32.3285567Z   install: false
2025-02-12T05:44:32.3286386Z   use: true
2025-02-12T05:44:32.3287182Z   cache-binary: true
2025-02-12T05:44:32.3288065Z   cleanup: true
2025-02-12T05:44:32.3288865Z env:
2025-02-12T05:44:32.3289759Z   IMAGE_TAG: ***/nginx
2025-02-12T05:44:32.3290642Z ##[endgroup]
2025-02-12T05:44:32.5933316Z ##[group]Docker info
2025-02-12T05:44:32.5971286Z [command]/usr/bin/docker version
2025-02-12T05:44:32.6566044Z Client: Docker Engine - Community
2025-02-12T05:44:32.6569025Z  Version:           26.1.3
2025-02-12T05:44:32.6570505Z  API version:       1.45
2025-02-12T05:44:32.6572142Z  Go version:        go1.21.10
2025-02-12T05:44:32.6573699Z  Git commit:        b72abbb
2025-02-12T05:44:32.6575266Z  Built:             Thu May 16 08:33:35 2024
2025-02-12T05:44:32.6577121Z  OS/Arch:           linux/amd64
2025-02-12T05:44:32.6578823Z  Context:           default
2025-02-12T05:44:32.6579842Z 
2025-02-12T05:44:32.6580495Z Server: Docker Engine - Community
2025-02-12T05:44:32.6582290Z  Engine:
2025-02-12T05:44:32.6583586Z   Version:          26.1.3
2025-02-12T05:44:32.6585277Z   API version:      1.45 (minimum version 1.24)
2025-02-12T05:44:32.6587220Z   Go version:       go1.21.10
2025-02-12T05:44:32.6588868Z   Git commit:       8e96db1
2025-02-12T05:44:32.6590503Z   Built:            Thu May 16 08:33:35 2024
2025-02-12T05:44:32.6592614Z   OS/Arch:          linux/amd64
2025-02-12T05:44:32.6594312Z   Experimental:     false
2025-02-12T05:44:32.6595899Z  containerd:
2025-02-12T05:44:32.6597287Z   Version:          1.7.25
2025-02-12T05:44:32.6599197Z   GitCommit:        bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
2025-02-12T05:44:32.6601273Z  runc:
2025-02-12T05:44:32.6602801Z   Version:          1.2.4
2025-02-12T05:44:32.6604469Z   GitCommit:        v1.2.4-0-g6c52b3f
2025-02-12T05:44:32.6606240Z  docker-init:
2025-02-12T05:44:32.6607589Z   Version:          0.19.0
2025-02-12T05:44:32.6609162Z   GitCommit:        de40ad0
2025-02-12T05:44:32.6628157Z [command]/usr/bin/docker info
2025-02-12T05:44:32.8541143Z Client: Docker Engine - Community
2025-02-12T05:44:32.8550627Z  Version:    26.1.3
2025-02-12T05:44:32.8552154Z  Context:    default
2025-02-12T05:44:32.8553531Z  Debug Mode: false
2025-02-12T05:44:32.8554834Z  Plugins:
2025-02-12T05:44:32.8556389Z   buildx: Docker Buildx (Docker Inc.)
2025-02-12T05:44:32.8557639Z     Version:  v0.20.1
2025-02-12T05:44:32.8558727Z     Path:     /usr/libexec/docker/cli-plugins/docker-buildx
2025-02-12T05:44:32.8560172Z   compose: Docker Compose (Docker Inc.)
2025-02-12T05:44:32.8561338Z     Version:  v2.27.1
2025-02-12T05:44:32.8562671Z     Path:     /usr/libexec/docker/cli-plugins/docker-compose
2025-02-12T05:44:32.8563681Z 
2025-02-12T05:44:32.8564016Z Server:
2025-02-12T05:44:32.8564774Z  Containers: 0
2025-02-12T05:44:32.8565610Z   Running: 0
2025-02-12T05:44:32.8566396Z   Paused: 0
2025-02-12T05:44:32.8567163Z   Stopped: 0
2025-02-12T05:44:32.8567923Z  Images: 0
2025-02-12T05:44:32.8568689Z  Server Version: 26.1.3
2025-02-12T05:44:32.8569581Z  Storage Driver: overlay2
2025-02-12T05:44:32.8570591Z   Backing Filesystem: extfs
2025-02-12T05:44:32.8571542Z   Supports d_type: true
2025-02-12T05:44:32.8572630Z   Using metacopy: false
2025-02-12T05:44:32.8573528Z   Native Overlay Diff: false
2025-02-12T05:44:32.8574480Z   userxattr: false
2025-02-12T05:44:32.8575321Z  Logging Driver: json-file
2025-02-12T05:44:32.8576363Z  Cgroup Driver: systemd
2025-02-12T05:44:32.8577242Z  Cgroup Version: 2
2025-02-12T05:44:32.8578053Z  Plugins:
2025-02-12T05:44:32.8579129Z   Volume: local
2025-02-12T05:44:32.8580551Z   Network: bridge host ipvlan macvlan null overlay
2025-02-12T05:44:32.8582511Z   Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
2025-02-12T05:44:32.8583973Z  Swarm: inactive
2025-02-12T05:44:32.8585184Z  Runtimes: io.containerd.runc.v2 runc
2025-02-12T05:44:32.8586427Z  Default Runtime: runc
2025-02-12T05:44:32.8587329Z  Init Binary: docker-init
2025-02-12T05:44:32.8588510Z  containerd version: bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
2025-02-12T05:44:32.8589820Z  runc version: v1.2.4-0-g6c52b3f
2025-02-12T05:44:32.8590794Z  init version: de40ad0
2025-02-12T05:44:32.8591685Z  Security Options:
2025-02-12T05:44:32.8592780Z   apparmor
2025-02-12T05:44:32.8593529Z   seccomp
2025-02-12T05:44:32.8594293Z    Profile: builtin
2025-02-12T05:44:32.8595115Z   cgroupns
2025-02-12T05:44:32.8595912Z  Kernel Version: 6.8.0-1021-azure
2025-02-12T05:44:32.8596957Z  Operating System: Ubuntu 24.04.1 LTS
2025-02-12T05:44:32.8597987Z  OSType: linux
2025-02-12T05:44:32.8598796Z  Architecture: x86_64
2025-02-12T05:44:32.8599642Z  CPUs: 4
2025-02-12T05:44:32.8600576Z  Total Memory: 15.62GiB
2025-02-12T05:44:32.8601622Z  Name: fv-az1371-407
2025-02-12T05:44:32.8602842Z  ID: c134ce27-11c9-4288-a10a-13dadbadd510
2025-02-12T05:44:32.8603975Z  Docker Root Dir: /var/lib/docker
2025-02-12T05:44:32.8604984Z  Debug Mode: false
2025-02-12T05:44:32.8605844Z  Username: githubactions
2025-02-12T05:44:32.8606770Z  Experimental: false
2025-02-12T05:44:32.8607632Z  Insecure Registries:
2025-02-12T05:44:32.8608479Z   127.0.0.0/8
2025-02-12T05:44:32.8609284Z  Live Restore Enabled: false
2025-02-12T05:44:32.8609884Z 
2025-02-12T05:44:32.8611090Z ##[endgroup]
2025-02-12T05:44:32.9101709Z ##[group]Buildx version
2025-02-12T05:44:32.9127262Z [command]/usr/bin/docker buildx version
2025-02-12T05:44:32.9553954Z github.com/docker/buildx v0.20.1 245093b99ab74aa2c729a496759afca0704d6470
2025-02-12T05:44:32.9585876Z ##[endgroup]
2025-02-12T05:44:32.9728390Z ##[group]Inspecting default docker context
2025-02-12T05:44:32.9880550Z [
2025-02-12T05:44:32.9882138Z   {
2025-02-12T05:44:32.9883423Z     "Name": "default",
2025-02-12T05:44:32.9884963Z     "Metadata": {},
2025-02-12T05:44:32.9886398Z     "Endpoints": {
2025-02-12T05:44:32.9887856Z       "docker": {
2025-02-12T05:44:32.9889464Z         "Host": "unix:///var/run/docker.sock",
2025-02-12T05:44:32.9891495Z         "SkipTLSVerify": false
2025-02-12T05:44:32.9893375Z       }
2025-02-12T05:44:32.9894661Z     },
2025-02-12T05:44:32.9895999Z     "TLSMaterial": {},
2025-02-12T05:44:32.9897532Z     "Storage": {
2025-02-12T05:44:32.9899045Z       "MetadataPath": "<IN MEMORY>",
2025-02-12T05:44:32.9900869Z       "TLSPath": "<IN MEMORY>"
2025-02-12T05:44:32.9902823Z     }
2025-02-12T05:44:32.9904323Z   }
2025-02-12T05:44:32.9905920Z ]
2025-02-12T05:44:32.9908268Z ##[endgroup]
2025-02-12T05:44:32.9910653Z ##[group]Creating a new builder instance
2025-02-12T05:44:33.0439708Z [command]/usr/bin/docker buildx create --name builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8 --driver docker-container --buildkitd-flags --allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host --use
2025-02-12T05:44:33.0954654Z builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8
2025-02-12T05:44:33.0987310Z ##[endgroup]
2025-02-12T05:44:33.0988820Z ##[group]Booting builder
2025-02-12T05:44:33.1039287Z [command]/usr/bin/docker buildx inspect --bootstrap --builder builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8
2025-02-12T05:44:33.1466985Z #1 [internal] booting buildkit
2025-02-12T05:44:33.2971402Z #1 pulling image moby/buildkit:buildx-stable-1
2025-02-12T05:44:35.2623145Z #1 pulling image moby/buildkit:buildx-stable-1 2.1s done
2025-02-12T05:44:35.4132409Z #1 creating container buildx_buildkit_builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d80
2025-02-12T05:44:35.5612360Z #1 creating container buildx_buildkit_builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d80 0.3s done
2025-02-12T05:44:35.5704095Z #1 DONE 2.4s
2025-02-12T05:44:35.5968468Z Name:          builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8
2025-02-12T05:44:35.5969528Z Driver:        docker-container
2025-02-12T05:44:35.5970170Z Last Activity: 2025-02-12 05:44:33 +0000 UTC
2025-02-12T05:44:35.5971296Z 
2025-02-12T05:44:35.5971558Z Nodes:
2025-02-12T05:44:35.5972389Z Name:                  builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d80
2025-02-12T05:44:35.5973844Z Endpoint:              unix:///var/run/docker.sock
2025-02-12T05:44:35.5974438Z Status:                running
2025-02-12T05:44:35.5975004Z BuildKit daemon flags: --allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host
2025-02-12T05:44:35.5975561Z BuildKit version:      v0.19.0
2025-02-12T05:44:35.5975917Z Platforms:             linux/amd64, linux/amd64/v2, linux/amd64/v3, linux/386
2025-02-12T05:44:35.5976357Z Labels:
2025-02-12T05:44:35.5976632Z  org.mobyproject.buildkit.worker.executor:         oci
2025-02-12T05:44:35.5977059Z  org.mobyproject.buildkit.worker.hostname:         84eee9c98faf
2025-02-12T05:44:35.5977514Z  org.mobyproject.buildkit.worker.network:          host
2025-02-12T05:44:35.5977926Z  org.mobyproject.buildkit.worker.oci.process-mode: sandbox
2025-02-12T05:44:35.5978346Z  org.mobyproject.buildkit.worker.selinux.enabled:  false
2025-02-12T05:44:35.5978758Z  org.mobyproject.buildkit.worker.snapshotter:      overlayfs
2025-02-12T05:44:35.5979116Z GC Policy rule#0:
2025-02-12T05:44:35.5979330Z  All:            false
2025-02-12T05:44:35.5979672Z  Filters:        type==source.local,type==exec.cachemount,type==source.git.checkout
2025-02-12T05:44:35.5980070Z  Keep Duration:  48h0m0s
2025-02-12T05:44:35.5980297Z  Max Used Space: 488.3MiB
2025-02-12T05:44:35.5980521Z GC Policy rule#1:
2025-02-12T05:44:35.5980736Z  All:            false
2025-02-12T05:44:35.5981037Z  Keep Duration:  1440h0m0s
2025-02-12T05:44:35.5981289Z  Reserved Space: 7.451GiB
2025-02-12T05:44:35.5981546Z  Max Used Space: 54.02GiB
2025-02-12T05:44:35.5981771Z  Min Free Space: 13.97GiB
2025-02-12T05:44:35.5982224Z GC Policy rule#2:
2025-02-12T05:44:35.5982443Z  All:            false
2025-02-12T05:44:35.5982658Z  Reserved Space: 7.451GiB
2025-02-12T05:44:35.5982879Z  Max Used Space: 54.02GiB
2025-02-12T05:44:35.5983103Z  Min Free Space: 13.97GiB
2025-02-12T05:44:35.5983317Z GC Policy rule#3:
2025-02-12T05:44:35.5983524Z  All:            true
2025-02-12T05:44:35.5983745Z  Reserved Space: 7.451GiB
2025-02-12T05:44:35.5983970Z  Max Used Space: 54.02GiB
2025-02-12T05:44:35.5984187Z  Min Free Space: 13.97GiB
2025-02-12T05:44:35.6016722Z ##[endgroup]
2025-02-12T05:44:35.6810012Z ##[group]Inspect builder
2025-02-12T05:44:35.6878181Z {
2025-02-12T05:44:35.6878506Z   "nodes": [
2025-02-12T05:44:35.6878724Z     {
2025-02-12T05:44:35.6879014Z       "name": "builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d80",
2025-02-12T05:44:35.6879411Z       "endpoint": "unix:///var/run/docker.sock",
2025-02-12T05:44:35.6879734Z       "status": "running",
2025-02-12T05:44:35.6880293Z       "buildkitd-flags": "--allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host",
2025-02-12T05:44:35.6880843Z       "buildkit": "v0.19.0",
2025-02-12T05:44:35.6881190Z       "platforms": "linux/amd64,linux/amd64/v2,linux/amd64/v3,linux/386",
2025-02-12T05:44:35.6881556Z       "features": {
2025-02-12T05:44:35.6882136Z         "Automatically load images to the Docker Engine image store": true,
2025-02-12T05:44:35.6882692Z         "Cache export": true,
2025-02-12T05:44:35.6882959Z         "Docker exporter": true,
2025-02-12T05:44:35.6883243Z         "Multi-platform build": true,
2025-02-12T05:44:35.6883667Z         "OCI exporter": true
2025-02-12T05:44:35.6883918Z       },
2025-02-12T05:44:35.6884141Z       "labels": {
2025-02-12T05:44:35.6884428Z         "org.mobyproject.buildkit.worker.executor": "oci",
2025-02-12T05:44:35.6884858Z         "org.mobyproject.buildkit.worker.hostname": "84eee9c98faf",
2025-02-12T05:44:35.6885277Z         "org.mobyproject.buildkit.worker.network": "host",
2025-02-12T05:44:35.6886096Z         "org.mobyproject.buildkit.worker.oci.process-mode": "sandbox",
2025-02-12T05:44:35.6886656Z         "org.mobyproject.buildkit.worker.selinux.enabled": "false",
2025-02-12T05:44:35.6887121Z         "org.mobyproject.buildkit.worker.snapshotter": "overlayfs"
2025-02-12T05:44:35.6887476Z       },
2025-02-12T05:44:35.6887674Z       "gcPolicy": [
2025-02-12T05:44:35.6888216Z         {
2025-02-12T05:44:35.6888537Z           "all": false,
2025-02-12T05:44:35.6888763Z           "filter": [
2025-02-12T05:44:35.6888997Z             "type==source.local",
2025-02-12T05:44:35.6889267Z             "type==exec.cachemount",
2025-02-12T05:44:35.6889552Z             "type==source.git.checkout"
2025-02-12T05:44:35.6889814Z           ],
2025-02-12T05:44:35.6890032Z           "keepDuration": "48h0m0s"
2025-02-12T05:44:35.6890284Z         },
2025-02-12T05:44:35.6890472Z         {
2025-02-12T05:44:35.6890667Z           "all": false,
2025-02-12T05:44:35.6890905Z           "keepDuration": "1440h0m0s"
2025-02-12T05:44:35.6891161Z         },
2025-02-12T05:44:35.6891348Z         {
2025-02-12T05:44:35.6891536Z           "all": false
2025-02-12T05:44:35.6891750Z         },
2025-02-12T05:44:35.6892180Z         {
2025-02-12T05:44:35.6892393Z           "all": true
2025-02-12T05:44:35.6892606Z         }
2025-02-12T05:44:35.6892792Z       ]
2025-02-12T05:44:35.6892995Z     }
2025-02-12T05:44:35.6893175Z   ],
2025-02-12T05:44:35.6893436Z   "name": "builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8",
2025-02-12T05:44:35.6893774Z   "driver": "docker-container",
2025-02-12T05:44:35.6894050Z   "lastActivity": "2025-02-12T05:44:33.000Z"
2025-02-12T05:44:35.6894312Z }
2025-02-12T05:44:35.6894750Z ##[endgroup]
2025-02-12T05:44:35.6895116Z ##[group]BuildKit version
2025-02-12T05:44:35.6895448Z builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d80: v0.19.0
2025-02-12T05:44:35.6895895Z ##[endgroup]
2025-02-12T05:44:35.7024802Z ##[group]Run docker/login-action@v3
2025-02-12T05:44:35.7025124Z with:
2025-02-12T05:44:35.7025561Z   username: ***
2025-02-12T05:44:35.7025844Z   password: ***
2025-02-12T05:44:35.7026066Z   ecr: auto
2025-02-12T05:44:35.7026253Z   logout: true
2025-02-12T05:44:35.7026443Z env:
2025-02-12T05:44:35.7026654Z   IMAGE_TAG: ***/nginx
2025-02-12T05:44:35.7026877Z ##[endgroup]
2025-02-12T05:44:35.9849810Z Logging into Docker Hub...
2025-02-12T05:44:36.2327188Z Login Succeeded!
2025-02-12T05:44:36.2440764Z ##[group]Run echo "GITHUB_REF: ${GITHUB_REF}"
2025-02-12T05:44:36.2441160Z [36;1mecho "GITHUB_REF: ${GITHUB_REF}"[0m
2025-02-12T05:44:36.2441490Z [36;1mif [[ "${GITHUB_REF}" == refs/tags/* ]]; then[0m
2025-02-12T05:44:36.2441797Z [36;1m  VERSION=${GITHUB_REF#refs/tags/}[0m
2025-02-12T05:44:36.2442304Z [36;1melse[0m
2025-02-12T05:44:36.2442656Z [36;1m  VERSION=$(git log -1 --pretty=format:%B | grep -Eo '[0-9]+\.[0-9]+\.[0-9]+' || echo "")[0m
2025-02-12T05:44:36.2443055Z [36;1mfi[0m
2025-02-12T05:44:36.2443261Z [36;1mif [[ -z "$VERSION" ]]; then[0m
2025-02-12T05:44:36.2443579Z [36;1m  echo "No version found in the commit message or tag"[0m
2025-02-12T05:44:36.2443876Z [36;1m  exit 1[0m
2025-02-12T05:44:36.2444062Z [36;1mfi[0m
2025-02-12T05:44:36.2444313Z [36;1mVERSION=${VERSION//[[:space:]]/}  # Remove any spaces[0m
2025-02-12T05:44:36.2444638Z [36;1mecho "Using version: $VERSION"[0m
2025-02-12T05:44:36.2444961Z [36;1mecho "VERSION=${VERSION}" >> $GITHUB_ENV[0m
2025-02-12T05:44:36.2620478Z shell: /usr/bin/bash -e {0}
2025-02-12T05:44:36.2620738Z env:
2025-02-12T05:44:36.2621005Z   IMAGE_TAG: ***/nginx
2025-02-12T05:44:36.2621232Z ##[endgroup]
2025-02-12T05:44:36.2718540Z GITHUB_REF: refs/heads/main
2025-02-12T05:44:36.2745501Z Using version: 1.0.3
2025-02-12T05:44:36.2861514Z ##[group]Run docker/build-push-action@v5
2025-02-12T05:44:36.2861827Z with:
2025-02-12T05:44:36.2862200Z   context: .
2025-02-12T05:44:36.2862397Z   file: ./Dockerfile
2025-02-12T05:44:36.2862606Z   push: true
2025-02-12T05:44:36.2862839Z   tags: ***/nginx:1.0.3
2025-02-12T05:44:36.2863062Z   load: false
2025-02-12T05:44:36.2863250Z   no-cache: false
2025-02-12T05:44:36.2863452Z   pull: false
2025-02-12T05:44:36.2863758Z   github-token: ***
2025-02-12T05:44:36.2863964Z env:
2025-02-12T05:44:36.2864162Z   IMAGE_TAG: ***/nginx
2025-02-12T05:44:36.2864379Z   VERSION: 1.0.3
2025-02-12T05:44:36.2864579Z ##[endgroup]
2025-02-12T05:44:36.5077738Z ##[group]GitHub Actions runtime token ACs
2025-02-12T05:44:36.5085106Z refs/heads/main: read/write
2025-02-12T05:44:36.5086278Z ##[endgroup]
2025-02-12T05:44:36.5086862Z ##[group]Docker info
2025-02-12T05:44:36.5125973Z [command]/usr/bin/docker version
2025-02-12T05:44:36.5320319Z Client: Docker Engine - Community
2025-02-12T05:44:36.5323585Z  Version:           26.1.3
2025-02-12T05:44:36.5324001Z  API version:       1.45
2025-02-12T05:44:36.5324372Z  Go version:        go1.21.10
2025-02-12T05:44:36.5324757Z  Git commit:        b72abbb
2025-02-12T05:44:36.5325156Z  Built:             Thu May 16 08:33:35 2024
2025-02-12T05:44:36.5325596Z  OS/Arch:           linux/amd64
2025-02-12T05:44:36.5325968Z  Context:           default
2025-02-12T05:44:36.5326199Z 
2025-02-12T05:44:36.5326354Z Server: Docker Engine - Community
2025-02-12T05:44:36.5326740Z  Engine:
2025-02-12T05:44:36.5327033Z   Version:          26.1.3
2025-02-12T05:44:36.5327439Z   API version:      1.45 (minimum version 1.24)
2025-02-12T05:44:36.5327908Z   Go version:       go1.21.10
2025-02-12T05:44:36.5328311Z   Git commit:       8e96db1
2025-02-12T05:44:36.5328696Z   Built:            Thu May 16 08:33:35 2024
2025-02-12T05:44:36.5329131Z   OS/Arch:          linux/amd64
2025-02-12T05:44:36.5329520Z   Experimental:     false
2025-02-12T05:44:36.5329884Z  containerd:
2025-02-12T05:44:36.5330202Z   Version:          1.7.25
2025-02-12T05:44:36.5330676Z   GitCommit:        bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
2025-02-12T05:44:36.5331191Z  runc:
2025-02-12T05:44:36.5331486Z   Version:          1.2.4
2025-02-12T05:44:36.5331848Z   GitCommit:        v1.2.4-0-g6c52b3f
2025-02-12T05:44:36.5332681Z  docker-init:
2025-02-12T05:44:36.5332935Z   Version:          0.19.0
2025-02-12T05:44:36.5333181Z   GitCommit:        de40ad0
2025-02-12T05:44:36.5369615Z [command]/usr/bin/docker info
2025-02-12T05:44:36.5819942Z Client: Docker Engine - Community
2025-02-12T05:44:36.5821124Z  Version:    26.1.3
2025-02-12T05:44:36.5823224Z  Context:    default
2025-02-12T05:44:36.5823698Z  Debug Mode: false
2025-02-12T05:44:36.5826103Z  Plugins:
2025-02-12T05:44:36.5826575Z   buildx: Docker Buildx (Docker Inc.)
2025-02-12T05:44:36.5827037Z     Version:  v0.20.1
2025-02-12T05:44:36.5827510Z     Path:     /usr/libexec/docker/cli-plugins/docker-buildx
2025-02-12T05:44:36.5828100Z   compose: Docker Compose (Docker Inc.)
2025-02-12T05:44:36.5828610Z     Version:  v2.27.1
2025-02-12T05:44:36.5829117Z     Path:     /usr/libexec/docker/cli-plugins/docker-compose
2025-02-12T05:44:36.5829600Z 
2025-02-12T05:44:36.5829729Z Server:
2025-02-12T05:44:36.5830036Z  Containers: 1
2025-02-12T05:44:36.5830360Z   Running: 1
2025-02-12T05:44:36.5830650Z   Paused: 0
2025-02-12T05:44:36.5830946Z   Stopped: 0
2025-02-12T05:44:36.5831267Z  Images: 1
2025-02-12T05:44:36.5831585Z  Server Version: 26.1.3
2025-02-12T05:44:36.5832170Z  Storage Driver: overlay2
2025-02-12T05:44:36.5832582Z   Backing Filesystem: extfs
2025-02-12T05:44:36.5833094Z   Supports d_type: true
2025-02-12T05:44:36.5833479Z   Using metacopy: false
2025-02-12T05:44:36.5833868Z   Native Overlay Diff: false
2025-02-12T05:44:36.5834279Z   userxattr: false
2025-02-12T05:44:36.5834643Z  Logging Driver: json-file
2025-02-12T05:44:36.5835046Z  Cgroup Driver: systemd
2025-02-12T05:44:36.5835425Z  Cgroup Version: 2
2025-02-12T05:44:36.5835771Z  Plugins:
2025-02-12T05:44:36.5836564Z   Volume: local
2025-02-12T05:44:36.5837010Z   Network: bridge host ipvlan macvlan null overlay
2025-02-12T05:44:36.5837708Z   Log: awslogs fluentd gcplogs gelf journald json-file local splunk syslog
2025-02-12T05:44:36.5838354Z  Swarm: inactive
2025-02-12T05:44:36.5838749Z  Runtimes: io.containerd.runc.v2 runc
2025-02-12T05:44:36.5839184Z  Default Runtime: runc
2025-02-12T05:44:36.5839564Z  Init Binary: docker-init
2025-02-12T05:44:36.5840094Z  containerd version: bcc810d6b9066471b0b6fa75f557a15a1cbf31bb
2025-02-12T05:44:36.5840683Z  runc version: v1.2.4-0-g6c52b3f
2025-02-12T05:44:36.5841099Z  init version: de40ad0
2025-02-12T05:44:36.5841456Z  Security Options:
2025-02-12T05:44:36.5841795Z   apparmor
2025-02-12T05:44:36.5842628Z   seccomp
2025-02-12T05:44:36.5842955Z    Profile: builtin
2025-02-12T05:44:36.5843313Z   cgroupns
2025-02-12T05:44:36.5843651Z  Kernel Version: 6.8.0-1021-azure
2025-02-12T05:44:36.5844110Z  Operating System: Ubuntu 24.04.1 LTS
2025-02-12T05:44:36.5844550Z  OSType: linux
2025-02-12T05:44:36.5844896Z  Architecture: x86_64
2025-02-12T05:44:36.5845252Z  CPUs: 4
2025-02-12T05:44:36.5845572Z  Total Memory: 15.62GiB
2025-02-12T05:44:36.5845934Z  Name: fv-az1371-407
2025-02-12T05:44:36.5846317Z  ID: c134ce27-11c9-4288-a10a-13dadbadd510
2025-02-12T05:44:36.5846795Z  Docker Root Dir: /var/lib/docker
2025-02-12T05:44:36.5847207Z  Debug Mode: false
2025-02-12T05:44:36.5847665Z  Username: ***
2025-02-12T05:44:36.5848024Z  Experimental: false
2025-02-12T05:44:36.5848404Z  Insecure Registries:
2025-02-12T05:44:36.5848754Z   127.0.0.0/8
2025-02-12T05:44:36.5849106Z  Live Restore Enabled: false
2025-02-12T05:44:36.5849372Z 
2025-02-12T05:44:36.5849980Z ##[endgroup]
2025-02-12T05:44:36.5850618Z ##[group]Proxy configuration
2025-02-12T05:44:36.5851081Z No proxy configuration found
2025-02-12T05:44:36.5851739Z ##[endgroup]
2025-02-12T05:44:36.6341193Z ##[group]Buildx version
2025-02-12T05:44:36.6366646Z [command]/usr/bin/docker buildx version
2025-02-12T05:44:36.6765581Z github.com/docker/buildx v0.20.1 245093b99ab74aa2c729a496759afca0704d6470
2025-02-12T05:44:36.6797811Z ##[endgroup]
2025-02-12T05:44:36.6798517Z ##[group]Builder info
2025-02-12T05:44:36.7614853Z {
2025-02-12T05:44:36.7615186Z   "nodes": [
2025-02-12T05:44:36.7615465Z     {
2025-02-12T05:44:36.7615732Z       "name": "builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d80",
2025-02-12T05:44:36.7616096Z       "endpoint": "unix:///var/run/docker.sock",
2025-02-12T05:44:36.7616375Z       "status": "running",
2025-02-12T05:44:36.7616870Z       "buildkitd-flags": "--allow-insecure-entitlement security.insecure --allow-insecure-entitlement network.host",
2025-02-12T05:44:36.7617374Z       "buildkit": "v0.19.0",
2025-02-12T05:44:36.7617696Z       "platforms": "linux/amd64,linux/amd64/v2,linux/amd64/v3,linux/386",
2025-02-12T05:44:36.7618063Z       "features": {
2025-02-12T05:44:36.7618370Z         "Automatically load images to the Docker Engine image store": true,
2025-02-12T05:44:36.7618738Z         "Cache export": true,
2025-02-12T05:44:36.7618998Z         "Docker exporter": true,
2025-02-12T05:44:36.7619252Z         "Multi-platform build": true,
2025-02-12T05:44:36.7619528Z         "OCI exporter": true
2025-02-12T05:44:36.7619750Z       },
2025-02-12T05:44:36.7619927Z       "labels": {
2025-02-12T05:44:36.7620187Z         "org.mobyproject.buildkit.worker.executor": "oci",
2025-02-12T05:44:36.7620576Z         "org.mobyproject.buildkit.worker.hostname": "84eee9c98faf",
2025-02-12T05:44:36.7620958Z         "org.mobyproject.buildkit.worker.network": "host",
2025-02-12T05:44:36.7621365Z         "org.mobyproject.buildkit.worker.oci.process-mode": "sandbox",
2025-02-12T05:44:36.7621797Z         "org.mobyproject.buildkit.worker.selinux.enabled": "false",
2025-02-12T05:44:36.7622428Z         "org.mobyproject.buildkit.worker.snapshotter": "overlayfs"
2025-02-12T05:44:36.7622743Z       },
2025-02-12T05:44:36.7622921Z       "gcPolicy": [
2025-02-12T05:44:36.7623113Z         {
2025-02-12T05:44:36.7623287Z           "all": false,
2025-02-12T05:44:36.7623495Z           "filter": [
2025-02-12T05:44:36.7623997Z             "type==source.local",
2025-02-12T05:44:36.7624265Z             "type==exec.cachemount",
2025-02-12T05:44:36.7624527Z             "type==source.git.checkout"
2025-02-12T05:44:36.7624773Z           ],
2025-02-12T05:44:36.7624965Z           "keepDuration": "48h0m0s"
2025-02-12T05:44:36.7625195Z         },
2025-02-12T05:44:36.7625364Z         {
2025-02-12T05:44:36.7625528Z           "all": false,
2025-02-12T05:44:36.7625737Z           "keepDuration": "1440h0m0s"
2025-02-12T05:44:36.7625968Z         },
2025-02-12T05:44:36.7626129Z         {
2025-02-12T05:44:36.7626309Z           "all": false
2025-02-12T05:44:36.7626503Z         },
2025-02-12T05:44:36.7626670Z         {
2025-02-12T05:44:36.7626996Z           "all": true
2025-02-12T05:44:36.7627188Z         }
2025-02-12T05:44:36.7627348Z       ]
2025-02-12T05:44:36.7627512Z     }
2025-02-12T05:44:36.7627667Z   ],
2025-02-12T05:44:36.7627901Z   "name": "builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8",
2025-02-12T05:44:36.7628219Z   "driver": "docker-container",
2025-02-12T05:44:36.7628476Z   "lastActivity": "2025-02-12T05:44:33.000Z"
2025-02-12T05:44:36.7628722Z }
2025-02-12T05:44:36.7629201Z ##[endgroup]
2025-02-12T05:44:36.9270101Z [command]/usr/bin/docker buildx build --file ./Dockerfile --iidfile /home/runner/work/_temp/docker-actions-toolkit-XzHX1c/build-iidfile-67c8a7e00c.txt --attest type=provenance,mode=max,builder-id=https://github.com/***/Test-application/actions/runs/13301267579 --tag ***/nginx:1.0.3 --metadata-file /home/runner/work/_temp/docker-actions-toolkit-XzHX1c/build-metadata-24e9d3650f.json --push .
2025-02-12T05:44:37.0247644Z #0 building with "builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8" instance using docker-container driver
2025-02-12T05:44:37.0248357Z 
2025-02-12T05:44:37.0248600Z #1 [internal] load build definition from Dockerfile
2025-02-12T05:44:37.2591756Z #1 transferring dockerfile: 99B done
2025-02-12T05:44:37.2592543Z #1 DONE 0.0s
2025-02-12T05:44:37.2592712Z 
2025-02-12T05:44:37.2592969Z #2 [auth] library/nginx:pull token for registry-1.docker.io
2025-02-12T05:44:37.2593413Z #2 DONE 0.0s
2025-02-12T05:44:37.2593574Z 
2025-02-12T05:44:37.2593810Z #3 [internal] load metadata for docker.io/library/nginx:1.27-alpine
2025-02-12T05:44:37.4807089Z #3 DONE 0.4s
2025-02-12T05:44:37.7020813Z 
2025-02-12T05:44:37.7021333Z #4 [internal] load .dockerignore
2025-02-12T05:44:37.7022100Z #4 transferring context: 2B done
2025-02-12T05:44:37.7023051Z #4 DONE 0.0s
2025-02-12T05:44:37.7023231Z 
2025-02-12T05:44:37.7023382Z #5 [internal] load build context
2025-02-12T05:44:37.7023794Z #5 transferring context: 135B done
2025-02-12T05:44:37.7024194Z #5 DONE 0.0s
2025-02-12T05:44:37.7024379Z 
2025-02-12T05:44:37.7025047Z #6 [1/2] FROM docker.io/library/nginx:1.27-alpine@sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef
2025-02-12T05:44:37.7025917Z #6 resolve docker.io/library/nginx:1.27-alpine@sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef done
2025-02-12T05:44:37.7061241Z #6 sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 2.10MB / 15.37MB 0.2s
2025-02-12T05:44:37.8094869Z #6 sha256:5281c445f8b7ca564e8124519f166a63c67a38074755a2d72738be7800dbbc78 1.40kB / 1.40kB 0.2s done
2025-02-12T05:44:37.8097976Z #6 sha256:4b56e0e1b50da94d716fc880809f96d5cb2751e0d9d9c680ae6c60e0f9239708 1.21kB / 1.21kB 0.2s done
2025-02-12T05:44:37.8101753Z #6 sha256:e92b9802c411990b7ff95ce238004213125726f1586227fe2665b765cd833602 404B / 404B 0.2s done
2025-02-12T05:44:37.8104893Z #6 sha256:9f41882e104d5e4b0e443fc387e440da00594e400a8c6c8d426e86a910e11084 956B / 956B 0.1s done
2025-02-12T05:44:37.8106035Z #6 sha256:f8813b38090d755304b9b62b790b0f19cd7eee409c3d0a3d495411be8eb35408 627B / 627B 0.1s done
2025-02-12T05:44:37.8112401Z #6 sha256:5215a08fb124932f41461b5da64ff183d8772c8cfa6d2ba64447f60275113f1b 1.79MB / 1.79MB 0.1s done
2025-02-12T05:44:37.9185554Z #6 sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 15.37MB / 15.37MB 0.3s done
2025-02-12T05:44:37.9188066Z #6 sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2ded61476c24403d592134f0 1.05MB / 3.64MB 0.2s
2025-02-12T05:44:38.0285718Z #6 sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2ded61476c24403d592134f0 3.64MB / 3.64MB 0.2s done
2025-02-12T05:44:38.0286555Z #6 extracting sha256:1f3e46996e2966e4faa5846e56e76e3748b7315e2ded61476c24403d592134f0 0.1s done
2025-02-12T05:44:38.0287095Z #6 DONE 0.5s
2025-02-12T05:44:38.1302831Z 
2025-02-12T05:44:38.1303849Z #6 [1/2] FROM docker.io/library/nginx:1.27-alpine@sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef
2025-02-12T05:44:38.1305335Z #6 extracting sha256:5215a08fb124932f41461b5da64ff183d8772c8cfa6d2ba64447f60275113f1b 0.1s done
2025-02-12T05:44:38.1306306Z #6 extracting sha256:f8813b38090d755304b9b62b790b0f19cd7eee409c3d0a3d495411be8eb35408 done
2025-02-12T05:44:38.1307177Z #6 extracting sha256:9f41882e104d5e4b0e443fc387e440da00594e400a8c6c8d426e86a910e11084 done
2025-02-12T05:44:38.1307682Z #6 DONE 0.6s
2025-02-12T05:44:38.2949118Z 
2025-02-12T05:44:38.2950258Z #6 [1/2] FROM docker.io/library/nginx:1.27-alpine@sha256:b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef
2025-02-12T05:44:38.2951468Z #6 extracting sha256:e92b9802c411990b7ff95ce238004213125726f1586227fe2665b765cd833602 done
2025-02-12T05:44:38.2952716Z #6 extracting sha256:4b56e0e1b50da94d716fc880809f96d5cb2751e0d9d9c680ae6c60e0f9239708 done
2025-02-12T05:44:38.2953484Z #6 extracting sha256:5281c445f8b7ca564e8124519f166a63c67a38074755a2d72738be7800dbbc78 done
2025-02-12T05:44:38.2954625Z #6 extracting sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413
2025-02-12T05:44:38.4018434Z #6 extracting sha256:a53100808f8943a9ff14b4843807b4141cca47498c4283d6aab09b76d9f77413 0.3s done
2025-02-12T05:44:38.5016954Z #6 DONE 0.9s
2025-02-12T05:44:38.5017201Z 
2025-02-12T05:44:38.5017385Z #7 [2/2] COPY index.html /usr/share/nginx/html
2025-02-12T05:44:38.5017858Z #7 DONE 0.1s
2025-02-12T05:44:38.5018055Z 
2025-02-12T05:44:38.5018215Z #8 exporting to image
2025-02-12T05:44:38.5018791Z #8 exporting layers 0.0s done
2025-02-12T05:44:38.5019397Z #8 exporting manifest sha256:6f9876a200b4176df30c0048f84efb0879c5ebf1666830a9fbc1948376a11123
2025-02-12T05:44:38.6946075Z #8 ...
2025-02-12T05:44:38.6946302Z 
2025-02-12T05:44:38.6946796Z #9 [auth] ***/nginx:pull,push token for registry-1.docker.io
2025-02-12T05:44:38.6947346Z #9 DONE 0.0s
2025-02-12T05:44:38.6947539Z 
2025-02-12T05:44:38.6947679Z #8 exporting to image
2025-02-12T05:44:38.6948389Z #8 exporting manifest sha256:6f9876a200b4176df30c0048f84efb0879c5ebf1666830a9fbc1948376a11123 done
2025-02-12T05:44:38.6949513Z #8 exporting config sha256:0582a7ee1b9be72cef57e8f8f1404b39df2bb1f1a8e7c6e6fbcda0a712626f55 done
2025-02-12T05:44:38.6950750Z #8 exporting attestation manifest sha256:c68dea603f7f9575014f193bdf0f3cc717fac1eb1edcafbd0e349c628df611ef done
2025-02-12T05:44:38.6952171Z #8 exporting manifest list sha256:04ea280329ece4104adaba3fb70c5812e59640ff879dcd43ccba341d7182dbb6 done
2025-02-12T05:44:38.6952701Z #8 pushing layers
2025-02-12T05:44:39.4636710Z #8 pushing layers 0.9s done
2025-02-12T05:44:39.4637699Z #8 pushing manifest for docker.io/***/nginx:1.0.3@sha256:04ea280329ece4104adaba3fb70c5812e59640ff879dcd43ccba341d7182dbb6
2025-02-12T05:44:40.6713476Z #8 pushing manifest for docker.io/***/nginx:1.0.3@sha256:04ea280329ece4104adaba3fb70c5812e59640ff879dcd43ccba341d7182dbb6 1.2s done
2025-02-12T05:44:40.6714218Z #8 DONE 2.2s
2025-02-12T05:44:40.6982766Z 
2025-02-12T05:44:40.6983210Z #10 resolving provenance for metadata file
2025-02-12T05:44:40.6983588Z #10 DONE 0.0s
2025-02-12T05:44:40.7062522Z ##[group]ImageID
2025-02-12T05:44:40.7063219Z sha256:04ea280329ece4104adaba3fb70c5812e59640ff879dcd43ccba341d7182dbb6
2025-02-12T05:44:40.7064234Z ##[endgroup]
2025-02-12T05:44:40.7064865Z ##[group]Digest
2025-02-12T05:44:40.7066585Z sha256:04ea280329ece4104adaba3fb70c5812e59640ff879dcd43ccba341d7182dbb6
2025-02-12T05:44:40.7067613Z ##[endgroup]
2025-02-12T05:44:40.7068218Z ##[group]Metadata
2025-02-12T05:44:40.7068590Z {
2025-02-12T05:44:40.7069218Z   "buildx.build.provenance": {
2025-02-12T05:44:40.7069619Z     "builder": {
2025-02-12T05:44:40.7070378Z       "id": "https://github.com/***/Test-application/actions/runs/13301267579"
2025-02-12T05:44:40.7070967Z     },
2025-02-12T05:44:40.7071351Z     "buildType": "https://mobyproject.org/buildkit@v1",
2025-02-12T05:44:40.7072135Z     "materials": [
2025-02-12T05:44:40.7072457Z       {
2025-02-12T05:44:40.7072872Z         "uri": "pkg:docker/nginx@1.27-alpine?platform=linux%2Famd64",
2025-02-12T05:44:40.7073341Z         "digest": {
2025-02-12T05:44:40.7073862Z           "sha256": "b471bb609adc83f73c2d95148cf1bd683408739a3c09c0afc666ea2af0037aef"
2025-02-12T05:44:40.7074457Z         }
2025-02-12T05:44:40.7074984Z       }
2025-02-12T05:44:40.7075262Z     ],
2025-02-12T05:44:40.7075549Z     "invocation": {
2025-02-12T05:44:40.7075892Z       "configSource": {},
2025-02-12T05:44:40.7076253Z       "parameters": {
2025-02-12T05:44:40.7076623Z         "frontend": "dockerfile.v0",
2025-02-12T05:44:40.7077043Z         "locals": [
2025-02-12T05:44:40.7077354Z           {
2025-02-12T05:44:40.7077646Z             "name": "context"
2025-02-12T05:44:40.7078013Z           },
2025-02-12T05:44:40.7078302Z           {
2025-02-12T05:44:40.7078597Z             "name": "dockerfile"
2025-02-12T05:44:40.7078970Z           }
2025-02-12T05:44:40.7079248Z         ]
2025-02-12T05:44:40.7079528Z       },
2025-02-12T05:44:40.7079830Z       "environment": {
2025-02-12T05:44:40.7080194Z         "platform": "linux/amd64"
2025-02-12T05:44:40.7080574Z       }
2025-02-12T05:44:40.7080852Z     }
2025-02-12T05:44:40.7081134Z   },
2025-02-12T05:44:40.7082199Z   "buildx.build.ref": "builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8/builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d80/gfeln99rdbh5975rsjaylmucr",
2025-02-12T05:44:40.7083221Z   "containerimage.descriptor": {
2025-02-12T05:44:40.7083746Z     "mediaType": "application/vnd.oci.image.index.v1+json",
2025-02-12T05:44:40.7084517Z     "digest": "sha256:04ea280329ece4104adaba3fb70c5812e59640ff879dcd43ccba341d7182dbb6",
2025-02-12T05:44:40.7085301Z     "size": 856
2025-02-12T05:44:40.7085610Z   },
2025-02-12T05:44:40.7086268Z   "containerimage.digest": "sha256:04ea280329ece4104adaba3fb70c5812e59640ff879dcd43ccba341d7182dbb6",
2025-02-12T05:44:40.7087115Z   "image.name": "***/nginx:1.0.3"
2025-02-12T05:44:40.7087499Z }
2025-02-12T05:44:40.7088034Z ##[endgroup]
2025-02-12T05:44:40.7204931Z Post job cleanup.
2025-02-12T05:44:40.9458342Z ##[group]Removing temp folder /home/runner/work/_temp/docker-actions-toolkit-XzHX1c
2025-02-12T05:44:40.9472487Z ##[endgroup]
2025-02-12T05:44:40.9473049Z ##[group]Post cache
2025-02-12T05:44:40.9474714Z State not set
2025-02-12T05:44:40.9475260Z ##[endgroup]
2025-02-12T05:44:40.9597002Z Post job cleanup.
2025-02-12T05:44:41.2459135Z [command]/usr/bin/docker logout 
2025-02-12T05:44:41.2584498Z Removing login credentials for https://index.docker.io/v1/
2025-02-12T05:44:41.2616524Z ##[group]Post cache
2025-02-12T05:44:41.2618285Z State not set
2025-02-12T05:44:41.2619637Z ##[endgroup]
2025-02-12T05:44:41.2740790Z Post job cleanup.
2025-02-12T05:44:41.5232289Z ##[group]Removing builder
2025-02-12T05:44:41.6165581Z [command]/usr/bin/docker buildx rm builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8
2025-02-12T05:44:41.8162529Z builder-1bd27bb1-d4f1-4de4-b810-7aa1819845d8 removed
2025-02-12T05:44:41.8199636Z ##[endgroup]
2025-02-12T05:44:41.8201240Z ##[group]Cleaning up certificates
2025-02-12T05:44:41.8207779Z ##[endgroup]
2025-02-12T05:44:41.8208360Z ##[group]Post cache
2025-02-12T05:44:41.8210514Z State not set
2025-02-12T05:44:41.8211530Z ##[endgroup]
2025-02-12T05:44:41.8330445Z Post job cleanup.
2025-02-12T05:44:41.9262333Z [command]/usr/bin/git version
2025-02-12T05:44:41.9300152Z git version 2.48.1
2025-02-12T05:44:41.9343718Z Temporarily overriding HOME='/home/runner/work/_temp/f7eb0dbf-16d0-4af0-9496-20d5ff76c196' before making global git config changes
2025-02-12T05:44:41.9344934Z Adding repository directory to the temporary git global config as a safe directory
2025-02-12T05:44:41.9349403Z [command]/usr/bin/git config --global --add safe.directory /home/runner/work/Test-application/Test-application
2025-02-12T05:44:41.9386940Z [command]/usr/bin/git config --local --name-only --get-regexp core\.sshCommand
2025-02-12T05:44:41.9421892Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'core\.sshCommand' && git config --local --unset-all 'core.sshCommand' || :"
2025-02-12T05:44:41.9661245Z [command]/usr/bin/git config --local --name-only --get-regexp http\.https\:\/\/github\.com\/\.extraheader
2025-02-12T05:44:41.9683266Z http.https://github.com/.extraheader
2025-02-12T05:44:41.9696395Z [command]/usr/bin/git config --local --unset-all http.https://github.com/.extraheader
2025-02-12T05:44:41.9728077Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'http\.https\:\/\/github\.com\/\.extraheader' && git config --local --unset-all 'http.https://github.com/.extraheader' || :"
2025-02-12T05:44:42.0067204Z Cleaning up orphan processes
```
</details>

<details>
  <summary>Лог выполнения Workflow file jobs deploy:</summary>
```
2025-02-12T05:44:32.9519021Z Current runner version: '2.319.1'
2025-02-12T05:44:32.9542776Z ##[group]Operating System
2025-02-12T05:44:32.9543722Z Ubuntu
2025-02-12T05:44:32.9544226Z 22.04.4
2025-02-12T05:44:32.9544823Z LTS
2025-02-12T05:44:32.9545416Z ##[endgroup]
2025-02-12T05:44:32.9545980Z ##[group]Runner Image
2025-02-12T05:44:32.9546667Z Image: ubuntu-22.04
2025-02-12T05:44:32.9547233Z Version: 20240811.1.0
2025-02-12T05:44:32.9548560Z Included Software: https://github.com/actions/runner-images/blob/ubuntu22/20240811.1/images/ubuntu/Ubuntu2204-Readme.md
2025-02-12T05:44:32.9550236Z Image Release: https://github.com/actions/runner-images/releases/tag/ubuntu22%2F20240811.1
2025-02-12T05:44:32.9551404Z ##[endgroup]
2025-02-12T05:44:32.9551983Z ##[group]Runner Image Provisioner
2025-02-12T05:44:32.9552751Z 2.0.374.1
2025-02-12T05:44:32.9553260Z ##[endgroup]
2025-02-12T05:44:32.9568491Z ##[group]GITHUB_TOKEN Permissions
2025-02-12T05:44:32.9570342Z Contents: read
2025-02-12T05:44:32.9571101Z Metadata: read
2025-02-12T05:44:32.9571920Z Packages: read
2025-02-12T05:44:32.9572587Z ##[endgroup]
2025-02-12T05:44:32.9575966Z Secret source: Actions
2025-02-12T05:44:32.9576730Z Prepare workflow directory
2025-02-12T05:44:33.0472605Z Prepare all required actions
2025-02-12T05:44:33.0628176Z Getting action download info
2025-02-12T05:44:33.1919105Z Download action repository 'actions/checkout@v4' (SHA:692973e3d937129bcbf40652eb9f2f61becf3332)
2025-02-12T05:44:33.3584226Z Complete job name: deploy
2025-02-12T05:44:33.4445840Z ##[group]Run actions/checkout@v4
2025-02-12T05:44:33.4446618Z with:
2025-02-12T05:44:33.4447382Z   repository: slava1005/Test-application
2025-02-12T05:44:33.4448473Z   token: ***
2025-02-12T05:44:33.4449055Z   ssh-strict: true
2025-02-12T05:44:33.4449733Z   ssh-*** git
2025-02-12T05:44:33.4450357Z   persist-credentials: true
2025-02-12T05:44:33.4451052Z   clean: true
2025-02-12T05:44:33.4451657Z   sparse-checkout-cone-mode: true
2025-02-12T05:44:33.4452421Z   fetch-depth: 1
2025-02-12T05:44:33.4453015Z   fetch-tags: false
2025-02-12T05:44:33.4453686Z   show-progress: true
2025-02-12T05:44:33.4454270Z   lfs: false
2025-02-12T05:44:33.4454879Z   submodules: false
2025-02-12T05:44:33.4455498Z   set-safe-directory: true
2025-02-12T05:44:33.4456176Z env:
2025-02-12T05:44:33.4456729Z   IMAGE_TAG: slava1005/nginx
2025-02-12T05:44:33.4734632Z   KUBECONFIG: ***
2025-02-12T05:44:33.4735370Z ##[endgroup]
2025-02-12T05:44:33.6693214Z Syncing repository: slava1005/Test-application
2025-02-12T05:44:33.6697127Z ##[group]Getting Git version info
2025-02-12T05:44:33.6699033Z Working directory is '/home/runner/work/Test-application/Test-application'
2025-02-12T05:44:33.6701797Z [command]/usr/bin/git version
2025-02-12T05:44:33.6703015Z git version 2.46.0
2025-02-12T05:44:33.6711255Z ##[endgroup]
2025-02-12T05:44:33.6729592Z Temporarily overriding HOME='/home/runner/work/_temp/a3f62ad1-d47c-4687-acc5-6f599471d8ee' before making global git config changes
2025-02-12T05:44:33.6732590Z Adding repository directory to the temporary git global config as a safe directory
2025-02-12T05:44:33.6741983Z [command]/usr/bin/git config --global --add safe.directory /home/runner/work/Test-application/Test-application
2025-02-12T05:44:33.6778351Z Deleting the contents of '/home/runner/work/Test-application/Test-application'
2025-02-12T05:44:33.6781374Z ##[group]Initializing the repository
2025-02-12T05:44:33.6785634Z [command]/usr/bin/git init /home/runner/work/Test-application/Test-application
2025-02-12T05:44:33.6844200Z hint: Using 'master' as the name for the initial branch. This default branch name
2025-02-12T05:44:33.6846168Z hint: is subject to change. To configure the initial branch name to use in all
2025-02-12T05:44:33.6848333Z hint: of your new repositories, which will suppress this warning, call:
2025-02-12T05:44:33.6849876Z hint:
2025-02-12T05:44:33.6850852Z hint: 	git config --global init.defaultBranch <name>
2025-02-12T05:44:33.6851772Z hint:
2025-02-12T05:44:33.6852742Z hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
2025-02-12T05:44:33.6854756Z hint: 'development'. The just-created branch can be renamed via this command:
2025-02-12T05:44:33.6855882Z hint:
2025-02-12T05:44:33.6856489Z hint: 	git branch -m <name>
2025-02-12T05:44:33.6858198Z Initialized empty Git repository in /home/runner/work/Test-application/Test-application/.git/
2025-02-12T05:44:33.6861253Z [command]/usr/bin/git remote add origin https://github.com/slava1005/Test-application
2025-02-12T05:44:33.6892063Z ##[endgroup]
2025-02-12T05:44:33.6893218Z ##[group]Disabling automatic garbage collection
2025-02-12T05:44:33.6895506Z [command]/usr/bin/git config --local gc.auto 0
2025-02-12T05:44:33.6926507Z ##[endgroup]
2025-02-12T05:44:33.6927757Z ##[group]Setting up auth
2025-02-12T05:44:33.6932697Z [command]/usr/bin/git config --local --name-only --get-regexp core\.sshCommand
2025-02-12T05:44:33.6964525Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'core\.sshCommand' && git config --local --unset-all 'core.sshCommand' || :"
2025-02-12T05:44:33.7248987Z [command]/usr/bin/git config --local --name-only --get-regexp http\.https\:\/\/github\.com\/\.extraheader
2025-02-12T05:44:33.7280766Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'http\.https\:\/\/github\.com\/\.extraheader' && git config --local --unset-all 'http.https://github.com/.extraheader' || :"
2025-02-12T05:44:33.7514614Z [command]/usr/bin/git config --local http.https://github.com/.extraheader AUTHORIZATION: basic ***
2025-02-12T05:44:33.7556796Z ##[endgroup]
2025-02-12T05:44:33.7557933Z ##[group]Fetching the repository
2025-02-12T05:44:33.7565172Z [command]/usr/bin/git -c protocol.version=2 fetch --no-tags --prune --no-recurse-submodules --depth=1 origin +a213082e79267fe9736e8f5db5e09b7d207f4eb1:refs/remotes/origin/main
2025-02-12T05:44:34.0197231Z From https://github.com/slava1005/Test-application
2025-02-12T05:44:34.0199526Z  * [new ref]         a213082e79267fe9736e8f5db5e09b7d207f4eb1 -> origin/main
2025-02-12T05:44:34.0222060Z ##[endgroup]
2025-02-12T05:44:34.0223801Z ##[group]Determining the checkout info
2025-02-12T05:44:34.0225177Z ##[endgroup]
2025-02-12T05:44:34.0229103Z [command]/usr/bin/git sparse-checkout disable
2025-02-12T05:44:34.0267177Z [command]/usr/bin/git config --local --unset-all extensions.worktreeConfig
2025-02-12T05:44:34.0294535Z ##[group]Checking out the ref
2025-02-12T05:44:34.0297963Z [command]/usr/bin/git checkout --progress --force -B main refs/remotes/origin/main
2025-02-12T05:44:34.0342237Z Switched to a new branch 'main'
2025-02-12T05:44:34.0345527Z branch 'main' set up to track 'origin/main'.
2025-02-12T05:44:34.0352321Z ##[endgroup]
2025-02-12T05:44:34.0387285Z [command]/usr/bin/git log -1 --format='%H'
2025-02-12T05:44:34.0410562Z 'a213082e79267fe9736e8f5db5e09b7d207f4eb1'
2025-02-12T05:44:34.0711436Z ##[group]Run curl -LO "https://dl.k8s.io/release/v1.30.3/bin/linux/amd64/kubectl"
2025-02-12T05:44:34.0713023Z �[36;1mcurl -LO "https://dl.k8s.io/release/v1.30.3/bin/linux/amd64/kubectl"�[0m
2025-02-12T05:44:34.0714249Z �[36;1mchmod +x ./kubectl�[0m
2025-02-12T05:44:34.0715082Z �[36;1msudo mv ./kubectl /usr/local/bin/kubectl�[0m
2025-02-12T05:44:34.0716039Z �[36;1mkubectl version --client�[0m
2025-02-12T05:44:34.0893783Z shell: /usr/bin/bash -e {0}
2025-02-12T05:44:34.0894665Z env:
2025-02-12T05:44:34.0895289Z   IMAGE_TAG: slava1005/nginx
2025-02-12T05:44:34.1223563Z   KUBECONFIG: ***
2025-02-12T05:44:34.1224293Z ##[endgroup]
2025-02-12T05:44:34.1458384Z   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
2025-02-12T05:44:34.1460865Z                                  Dload  Upload   Total   Spent    Left  Speed
2025-02-12T05:44:34.1461956Z 
2025-02-12T05:44:34.1983238Z   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
2025-02-12T05:44:34.1986484Z 100   138  100   138    0     0   2600      0 --:--:-- --:--:-- --:--:--  2653
2025-02-12T05:44:34.2512993Z 
2025-02-12T05:44:35.2501291Z   0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
2025-02-12T05:44:36.2510183Z  40 49.0M   40 19.6M    0     0  17.7M      0  0:00:02  0:00:01  0:00:01 19.6M
2025-02-12T05:44:36.7608870Z  79 49.0M   79 39.0M    0     0  18.5M      0  0:00:02  0:00:02 --:--:-- 19.5M
2025-02-12T05:44:36.7611201Z 100 49.0M  100 49.0M    0     0  18.7M      0  0:00:02  0:00:02 --:--:-- 19.5M
2025-02-12T05:44:36.8084832Z Client Version: v1.30.3
2025-02-12T05:44:36.8085909Z Kustomize Version: v5.0.4-0.20230601165947-6ce0bf390ce3
2025-02-12T05:44:36.8194032Z ##[group]Run echo "***
2025-02-12T05:44:36.8195000Z �[36;1mecho "***�[0m
2025-02-12T05:44:36.8195587Z �[36;1m***�[0m
2025-02-12T05:44:36.8196176Z �[36;1m***�[0m
2025-02-12T05:44:36.8230132Z �[36;1m    ***�[0m
2025-02-12T05:44:36.8230835Z �[36;1m    ***�[0m
2025-02-12T05:44:36.8231456Z �[36;1m  ***�[0m
2025-02-12T05:44:36.8231962Z �[36;1m***�[0m
2025-02-12T05:44:36.8232557Z �[36;1m***�[0m
2025-02-12T05:44:36.8233077Z �[36;1m    ***�[0m
2025-02-12T05:44:36.8233683Z �[36;1m    ***�[0m
2025-02-12T05:44:36.8234325Z �[36;1m  ***�[0m
2025-02-12T05:44:36.8235007Z �[36;1m***�[0m
2025-02-12T05:44:36.8235503Z �[36;1m***�[0m
2025-02-12T05:44:36.8236115Z �[36;1m***�[0m
2025-02-12T05:44:36.8236641Z �[36;1m***�[0m
2025-02-12T05:44:36.8237159Z �[36;1m***�[0m
2025-02-12T05:44:36.8237714Z �[36;1m  ***�[0m
2025-02-12T05:44:36.8274206Z �[36;1m    ***�[0m
2025-02-12T05:44:36.8346439Z �[36;1m    ***" > config.yml�[0m
2025-02-12T05:44:36.8347103Z �[36;1mexport KUBECONFIG=config.yml�[0m
2025-02-12T05:44:36.8347924Z �[36;1mkubectl config view�[0m
2025-02-12T05:44:36.8348590Z �[36;1mkubectl get nodes�[0m
2025-02-12T05:44:36.8349197Z �[36;1mkubectl get pods --all-namespaces�[0m
2025-02-12T05:44:36.8350090Z �[36;1mkubectl create deployment nginx --image=slava1005/nginx:1.0.3�[0m
2025-02-12T05:44:36.8350997Z �[36;1mkubectl rollout status deployment.v1.apps/nginx�[0m
2025-02-12T05:44:36.8415929Z shell: /usr/bin/bash -e {0}
2025-02-12T05:44:36.8416581Z env:
2025-02-12T05:44:36.8417323Z   IMAGE_TAG: slava1005/nginx
2025-02-12T05:44:36.8560203Z   KUBECONFIG: ***
2025-02-12T05:44:36.8560798Z ##[endgroup]
2025-02-12T05:44:36.8997024Z ***
2025-02-12T05:44:36.8998171Z ***
2025-02-12T05:44:36.8999521Z ***
2025-02-12T05:44:36.9000703Z     certificate-authority-data: DATA+OMITTED
2025-02-12T05:44:36.9002320Z     ***
2025-02-12T05:44:36.9003402Z   ***
2025-02-12T05:44:36.9004325Z ***
2025-02-12T05:44:36.9005404Z ***
2025-02-12T05:44:36.9006390Z     ***
2025-02-12T05:44:36.9007533Z     ***
2025-02-12T05:44:36.9008768Z   ***
2025-02-12T05:44:36.9010077Z ***
2025-02-12T05:44:36.9011021Z ***
2025-02-12T05:44:36.9012130Z ***
2025-02-12T05:44:36.9013010Z ***
2025-02-12T05:44:36.9014216Z ***
2025-02-12T05:44:36.9015340Z   ***
2025-02-12T05:44:36.9016226Z     client-certificate-data: DATA+OMITTED
2025-02-12T05:44:36.9017707Z     client-key-data: DATA+OMITTED
2025-02-12T05:44:37.7965463Z NAME       STATUS   ROLES           AGE    VERSION
2025-02-12T05:44:37.7967674Z master-1   Ready    control-plane   144m   v1.30.3
2025-02-12T05:44:37.7969081Z worker-1   Ready    <none>          143m   v1.30.3
2025-02-12T05:44:37.7970395Z worker-2   Ready    <none>          143m   v1.30.3
2025-02-12T05:44:38.4197657Z NAMESPACE     NAME                                      READY   STATUS    RESTARTS       AGE
2025-02-12T05:44:38.4199578Z application   nginx-deamonset-f8wht                     1/1     Running   0              117m
2025-02-12T05:44:38.4201614Z application   nginx-deamonset-hbh6z                     1/1     Running   0              117m
2025-02-12T05:44:38.4203716Z kube-system   calico-kube-controllers-c7cc688f8-f6ldp   1/1     Running   0              143m
2025-02-12T05:44:38.4205658Z kube-system   calico-node-6vhk8                         1/1     Running   0              143m
2025-02-12T05:44:38.4207475Z kube-system   calico-node-cfm7w                         1/1     Running   0              143m
2025-02-12T05:44:38.4209262Z kube-system   calico-node-n6gwb                         1/1     Running   0              143m
2025-02-12T05:44:38.4211490Z kube-system   coredns-776bb9db5d-82v8f                  1/1     Running   0              142m
2025-02-12T05:44:38.4213304Z kube-system   coredns-776bb9db5d-r6zkg                  1/1     Running   0              142m
2025-02-12T05:44:38.4217050Z kube-system   dns-autoscaler-6ffb84bd6-5t5f8            1/1     Running   0              142m
2025-02-12T05:44:38.4219274Z kube-system   kube-apiserver-master-1                   1/1     Running   0              144m
2025-02-12T05:44:38.4221916Z kube-system   kube-controller-manager-master-1          1/1     Running   1              144m
2025-02-12T05:44:38.4224138Z kube-system   kube-proxy-b99ts                          1/1     Running   0              143m
2025-02-12T05:44:38.4229113Z kube-system   kube-proxy-cddk2                          1/1     Running   0              143m
2025-02-12T05:44:38.4231125Z kube-system   kube-proxy-r5gk2                          1/1     Running   0              143m
2025-02-12T05:44:38.4235963Z kube-system   kube-scheduler-master-1                   1/1     Running   1              144m
2025-02-12T05:44:38.4238181Z kube-system   nginx-proxy-worker-1                      1/1     Running   0              143m
2025-02-12T05:44:38.4241039Z kube-system   nginx-proxy-worker-2                      1/1     Running   0              143m
2025-02-12T05:44:38.4243001Z kube-system   nodelocaldns-99qdh                        1/1     Running   2 (121m ago)   121m
2025-02-12T05:44:38.4245055Z kube-system   nodelocaldns-cgbvk                        1/1     Running   2 (121m ago)   121m
2025-02-12T05:44:38.4247013Z kube-system   nodelocaldns-xqr6g                        1/1     Running   0              121m
2025-02-12T05:44:38.4249016Z monitoring    alertmanager-main-0                       2/2     Running   0              138m
2025-02-12T05:44:38.4251039Z monitoring    alertmanager-main-1                       2/2     Running   0              138m
2025-02-12T05:44:38.4253041Z monitoring    alertmanager-main-2                       2/2     Running   0              138m
2025-02-12T05:44:38.4255124Z monitoring    blackbox-exporter-597d86cf5c-w8c2q        3/3     Running   0              139m
2025-02-12T05:44:38.4257547Z monitoring    grafana-549c49555-zg8tr                   1/1     Running   0              139m
2025-02-12T05:44:38.4259737Z monitoring    kube-state-metrics-789f4b647d-vcnwv       3/3     Running   0              139m
2025-02-12T05:44:38.4262840Z monitoring    node-exporter-dlg58                       2/2     Running   0              139m
2025-02-12T05:44:38.4264591Z monitoring    node-exporter-fwdqz                       2/2     Running   0              139m
2025-02-12T05:44:38.4266367Z monitoring    node-exporter-lsqgw                       2/2     Running   0              139m
2025-02-12T05:44:38.4268273Z monitoring    prometheus-adapter-5794d7d9f5-6q6n5       1/1     Running   0              139m
2025-02-12T05:44:38.4270267Z monitoring    prometheus-adapter-5794d7d9f5-xhl4r       1/1     Running   0              139m
2025-02-12T05:44:38.4271396Z monitoring    prometheus-k8s-0                          2/2     Running   0              138m
2025-02-12T05:44:38.4272521Z monitoring    prometheus-k8s-1                          2/2     Running   0              138m
2025-02-12T05:44:38.4273657Z monitoring    prometheus-operator-7d9bfb969b-lmsb6      2/2     Running   0              139m
2025-02-12T05:44:38.8923107Z deployment.apps/nginx created
2025-02-12T05:44:39.5039141Z Waiting for deployment "nginx" rollout to finish: 0 of 1 updated replicas are available...
2025-02-12T05:44:40.6604273Z deployment "nginx" successfully rolled out
2025-02-12T05:44:40.6719124Z Post job cleanup.
2025-02-12T05:44:40.7638027Z [command]/usr/bin/git version
2025-02-12T05:44:40.7674421Z git version 2.46.0
2025-02-12T05:44:40.7717014Z Temporarily overriding HOME='/home/runner/work/_temp/117a2fba-4224-4863-baf6-3f38a5c0d96f' before making global git config changes
2025-02-12T05:44:40.7719458Z Adding repository directory to the temporary git global config as a safe directory
2025-02-12T05:44:40.7730133Z [command]/usr/bin/git config --global --add safe.directory /home/runner/work/Test-application/Test-application
2025-02-12T05:44:40.7764308Z [command]/usr/bin/git config --local --name-only --get-regexp core\.sshCommand
2025-02-12T05:44:40.7797158Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'core\.sshCommand' && git config --local --unset-all 'core.sshCommand' || :"
2025-02-12T05:44:40.8034725Z [command]/usr/bin/git config --local --name-only --get-regexp http\.https\:\/\/github\.com\/\.extraheader
2025-02-12T05:44:40.8055327Z http.https://github.com/.extraheader
2025-02-12T05:44:40.8068300Z [command]/usr/bin/git config --local --unset-all http.https://github.com/.extraheader
2025-02-12T05:44:40.8100669Z [command]/usr/bin/git submodule foreach --recursive sh -c "git config --local --name-only --get-regexp 'http\.https\:\/\/github\.com\/\.extraheader' && git config --local --unset-all 'http.https://github.com/.extraheader' || :"
2025-02-12T05:44:40.8534255Z Cleaning up orphan processes
```
</details>
В свою очередь на docker hub появляется загруженный образ:

![img7](https://github.com/user-attachments/assets/8dd901af-5043-4d07-b58e-123b1689449a)


https://hub.docker.com/repository/docker/slava1005/nginx/general







