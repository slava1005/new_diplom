## Создание облачной инфраструктуры
```
Для начала необходимо подготовить облачную инфраструктуру в ЯО при помощи Terraform.

Особенности выполнения:

Бюджет купона ограничен, что следует иметь в виду при проектировании инфраструктуры и использовании ресурсов; Для облачного k8s используйте региональный мастер(неотказоустойчивый). Для self-hosted k8s минимизируйте ресурсы ВМ и долю ЦПУ. В обоих вариантах используйте прерываемые ВМ для worker nodes.
Предварительная подготовка к установке и запуску Kubernetes кластера.

Создайте сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами. Не стоит использовать права суперпользователя
Подготовьте backend для Terraform:
а. Рекомендуемый вариант: S3 bucket в созданном ЯО аккаунте(создание бакета через TF) б. Альтернативный вариант: Terraform Cloud
Создайте конфигурацию Terrafrom, используя созданный бакет ранее как бекенд для хранения стейт файла. Конфигурации Terraform для создания сервисного аккаунта и бакета и основной инфраструктуры следует сохранить в разных папках.
Создайте VPC с подсетями в разных зонах доступности.
Убедитесь, что теперь вы можете выполнить команды terraform destroy и terraform apply без дополнительных ручных действий.
В случае использования Terraform Cloud в качестве backend убедитесь, что применение изменений успешно проходит, используя web-интерфейс Terraform cloud.
Ожидаемые результаты:

Terraform сконфигурирован и создание инфраструктуры посредством Terraform возможно без дополнительных ручных действий, стейт основной конфигурации сохраняется в бакете или Terraform Cloud
Полученная конфигурация инфраструктуры является предварительной, поэтому в ходе дальнейшего выполнения задания возможны изменения.
```

## Решение:
### Подготовим облачную инфраструктуру в Яндекс.Облако при помощи Terraform.

### 1.1. Создадим сервисный аккаунт, который будет в дальнейшем использоваться Terraform для работы с инфраструктурой с необходимыми и достаточными правами.
```
# Создание сервисного аккаунта для Terraform
resource "yandex_iam_service_account" "service" {
  name      = var.account_name
  description = "service account to manage VMs"
  folder_id = var.folder_id
}

# Назначение роли editor сервисному аккаунту
resource "yandex_resourcemanager_folder_iam_member" "editor" {
  folder_id = var.folder_id
  role      = "editor"
  member    = "serviceAccount:${yandex_iam_service_account.service.id}"
  depends_on = [yandex_iam_service_account.service]
}

# Создание статического ключа доступа для сервисного аккаунта
resource "yandex_iam_service_account_static_access_key" "terraform_service_account_key" {
  service_account_id = yandex_iam_service_account.service.id
  description        = "static access key for object storage"
}
```
### 1.2. Подготовим backend для Terraform:
```
# Создадим бакет с использованием ключа
resource "yandex_storage_bucket" "state_storage" {
  bucket     = local.bucket_name
  access_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.secret_key

  anonymous_access_flags {
    read = false
    list = false
  }
}

# Локальная переменная отвечающая за текущую дату в названии бакета
locals {
    current_timestamp = timestamp()
    formatted_date = formatdate("DD-MM-YYYY", local.current_timestamp)
    bucket_name = "state-storage-${local.formatted_date}"
}

# Создание объекта в существующей папке
resource "yandex_storage_object" "backend" {
  access_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.access_key
  secret_key = yandex_iam_service_account_static_access_key.terraform_service_account_key.secret_key
  bucket = local.bucket_name
  key    = "terraform.tfstate"
  source = "./terraform.tfstate"
  depends_on = [yandex_storage_bucket.state_storage]
}
```
### 1.3. Создадим VPC с подсетями в разных зонах доступности.
```
#Создание пустой VPC
resource "yandex_vpc_network" "vpc0" {
  name = var.vpc_name
}

#Создадим в VPC subnet c названием subnet-a
resource "yandex_vpc_subnet" "subnet-a" {
  name           = var.subnet-a
  zone           = var.zone-a
  network_id     = yandex_vpc_network.vpc0.id
  v4_cidr_blocks = var.cidr-a
}

#Создание в VPC subnet с названием subnet-b
resource "yandex_vpc_subnet" "subnet-b" {
  name           = var.subnet-b
  zone           = var.zone-b
  network_id     = yandex_vpc_network.vpc0.id
  v4_cidr_blocks = var.cidr-b
}

#Создание в VPC subnet с названием subnet-d
resource "yandex_vpc_subnet" "subnet-d" {
  name           = var.subnet-d
  zone           = var.zone-d
  network_id     = yandex_vpc_network.vpc0.id
  v4_cidr_blocks = var.cidr-d
}

variable "vpc_name" {
  type        = string
  default     = "vpc0"
  description = "VPC network"
}

variable "subnet-a" {
  type        = string
  default     = "subnet-a"
  description = "subnet name"
}

variable "subnet-b" {
  type        = string
  default     = "subnet-b"
  description = "subnet name"
}

variable "subnet-d" {
  type        = string
  default     = "subnet-d"
  description = "subnet name"
}

variable "zone-a" {
  type        = string
  default     = "ru-central1-a"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "zone-b" {
  type        = string
  default     = "ru-central1-b"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "zone-d" {
  type        = string
  default     = "ru-central1-d"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "cidr-a" {
  type        = list(string)
  default     = ["10.0.1.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "cidr-b" {
  type        = list(string)
  default     = ["10.0.2.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "cidr-d" {
  type        = list(string)
  default     = ["10.0.3.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}
```

### 1.4. Убедимся, что теперь выполняется команды terraform apply без дополнительных ручных действий.
```
slava@DESKTOP-QKJU13U:~/new_diplom/schaffung_terraform/config/terraform$ terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # yandex_iam_service_account.service will be created
  + resource "yandex_iam_service_account" "service" {
      + created_at  = (known after apply)
      + description = "service account to manage VMs"
      + folder_id   = "b1gfurnb189rkavk8tlr"
      + id          = (known after apply)
      + name        = "savilov-vv"
    }

  # yandex_iam_service_account_static_access_key.terraform_service_account_key will be created
  + resource "yandex_iam_service_account_static_access_key" "terraform_service_account_key" {
      + access_key                   = (known after apply)
      + created_at                   = (known after apply)
      + description                  = "static access key for object storage"
      + encrypted_secret_key         = (known after apply)
      + id                           = (known after apply)
      + key_fingerprint              = (known after apply)
      + output_to_lockbox_version_id = (known after apply)
      + secret_key                   = (sensitive value)
      + service_account_id           = (known after apply)
    }

  # yandex_resourcemanager_folder_iam_member.editor will be created
  + resource "yandex_resourcemanager_folder_iam_member" "editor" {
      + folder_id = "b1gfurnb189rkavk8tlr"
      + id        = (known after apply)
      + member    = (known after apply)
      + role      = "editor"
    }

  # yandex_storage_bucket.state_storage will be created
  + resource "yandex_storage_bucket" "state_storage" {
      + access_key            = (known after apply)
      + bucket                = (known after apply)
      + bucket_domain_name    = (known after apply)
      + default_storage_class = (known after apply)
      + folder_id             = (known after apply)
      + force_destroy         = false
      + id                    = (known after apply)
      + secret_key            = (sensitive value)
      + website_domain        = (known after apply)
      + website_endpoint      = (known after apply)

      + anonymous_access_flags {
          + list = false
          + read = false
        }

      + versioning (known after apply)
    }

  # yandex_storage_object.backend will be created
  + resource "yandex_storage_object" "backend" {
      + access_key   = (known after apply)
      + acl          = "private"
      + bucket       = (known after apply)
      + content_type = (known after apply)
      + id           = (known after apply)
      + key          = "terraform.tfstate"
      + secret_key   = (sensitive value)
      + source       = "./terraform.tfstate"
    }

  # yandex_vpc_network.vpc0 will be created
  + resource "yandex_vpc_network" "vpc0" {
      + created_at                = (known after apply)
      + default_security_group_id = (known after apply)
      + folder_id                 = (known after apply)
      + id                        = (known after apply)
      + labels                    = (known after apply)
      + name                      = "vpc0"
      + subnet_ids                = (known after apply)
    }

  # yandex_vpc_subnet.subnet-a will be created
  + resource "yandex_vpc_subnet" "subnet-a" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet-a"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.0.1.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-a"
    }

  # yandex_vpc_subnet.subnet-b will be created
  + resource "yandex_vpc_subnet" "subnet-b" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet-b"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.0.2.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-b"
    }

  # yandex_vpc_subnet.subnet-d will be created
  + resource "yandex_vpc_subnet" "subnet-d" {
      + created_at     = (known after apply)
      + folder_id      = (known after apply)
      + id             = (known after apply)
      + labels         = (known after apply)
      + name           = "subnet-d"
      + network_id     = (known after apply)
      + v4_cidr_blocks = [
          + "10.0.3.0/24",
        ]
      + v6_cidr_blocks = (known after apply)
      + zone           = "ru-central1-d"
    }

Plan: 9 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

yandex_iam_service_account.service: Creating...
yandex_vpc_network.vpc0: Creating...
yandex_vpc_network.vpc0: Creation complete after 2s [id=enpute6go5jr46gaa61s]
yandex_vpc_subnet.subnet-a: Creating...
yandex_vpc_subnet.subnet-d: Creating...
yandex_vpc_subnet.subnet-b: Creating...
yandex_iam_service_account.service: Creation complete after 2s [id=aje78qiil5cavn81suc3]
yandex_resourcemanager_folder_iam_member.editor: Creating...
yandex_iam_service_account_static_access_key.terraform_service_account_key: Creating...
yandex_vpc_subnet.subnet-b: Creation complete after 1s [id=e2lbtbqqjqh6u648ct2a]
yandex_vpc_subnet.subnet-d: Creation complete after 1s [id=fl8gjsv74s1lflildtr8]
yandex_iam_service_account_static_access_key.terraform_service_account_key: Creation complete after 2s [id=ajeaq1vj2816b6fj59lp]
yandex_vpc_subnet.subnet-a: Creation complete after 2s [id=e9b9tiagbnkc6rmdcu8a]
yandex_storage_bucket.state_storage: Creating...
yandex_resourcemanager_folder_iam_member.editor: Creation complete after 3s [id=b1gfurnb189rkavk8tlr/editor/serviceAccount:aje78qiil5cavn81suc3]
yandex_storage_bucket.state_storage: Creation complete after 7s [id=state-storage1-11-02-2025]
yandex_storage_object.backend: Creating...
yandex_storage_object.backend: Creation complete after 0s [id=terraform.tfstate]

Apply complete! Resources: 9 added, 0 changed, 0 destroyed.
```

### Посмотрим созданные ресурсы с помощью CLI
```
slava@DESKTOP-QKJU13U:~/new_diplom/schaffung_terraform/config/terraform$ yc vpc network list
+----------------------+------+
|          ID          | NAME |
+----------------------+------+
| enpute6go5jr46gaa61s | vpc0 |
+----------------------+------+

slava@DESKTOP-QKJU13U:~/new_diplom/schaffung_terraform/config/terraform$ yc vpc subnet list
+----------------------+----------+----------------------+----------------+---------------+---------------+
|          ID          |   NAME   |      NETWORK ID      | ROUTE TABLE ID |     ZONE      |     RANGE     |
+----------------------+----------+----------------------+----------------+---------------+---------------+
| e2lbtbqqjqh6u648ct2a | subnet-b | enpute6go5jr46gaa61s |                | ru-central1-b | [10.0.2.0/24] |
| e9b9tiagbnkc6rmdcu8a | subnet-a | enpute6go5jr46gaa61s |                | ru-central1-a | [10.0.1.0/24] |
| fl8gjsv74s1lflildtr8 | subnet-d | enpute6go5jr46gaa61s |                | ru-central1-d | [10.0.3.0/24] |
+----------------------+----------+----------------------+----------------+---------------+---------------+

slava@DESKTOP-QKJU13U:~/new_diplom/schaffung_terraform/config/terraform$ yc storage bucket list
+---------------------------+----------------------+----------+-----------------------+---------------------+
|           NAME            |      FOLDER ID       | MAX SIZE | DEFAULT STORAGE CLASS |     CREATED AT      |
+---------------------------+----------------------+----------+-----------------------+---------------------+
| state-storage1-11-02-2025 | b1gfurnb189rkavk8tlr |        0 | STANDARD              | 2025-02-11 14:40:15 |
+---------------------------+----------------------+----------+-----------------------+---------------------+

slava@DESKTOP-QKJU13U:~/new_diplom/schaffung_terraform/config/terraform$ yc storage bucket stats --name state-storage1-11-02-2025
name: state-storage1-11-02-2025
default_storage_class: STANDARD
anonymous_access_flags:
  read: false
  list: false
  config_read: false
created_at: "2025-02-11T14:40:15.152096Z"
updated_at: "2025-02-11T14:40:15.152096Z"
```

### 1.5. Убедимся, что теперь выполняется команды terraform destroy без дополнительных ручных действий.
```
slava@DESKTOP-QKJU13U:~/new_diplom/schaffung_terraform/config/terraform$ terraform destroy

yandex_iam_service_account.service: Refreshing state... [id=aje9pcua6mbl3p4o44pk]
yandex_vpc_network.vpc0: Refreshing state... [id=enpifgf362inb6b5mf0j]
yandex_resourcemanager_folder_iam_member.editor: Refreshing state... [id=b1gfurnb189rkavk8tlr/editor/serviceAccount:aje9pcua6mbl3p4o44pk]
yandex_iam_service_account_static_access_key.terraform_service_account_key: Refreshing state... [id=ajeevp64o5h0f8t1c9c3]
yandex_storage_bucket.state_storage: Refreshing state... [id=state-storage1-11-02-2025]
yandex_vpc_subnet.subnet-d: Refreshing state... [id=fl8fvmss0rv12jr1siqb]
yandex_vpc_subnet.subnet-b: Refreshing state... [id=e2l2746r0p7qrq8v81bh]
yandex_vpc_subnet.subnet-a: Refreshing state... [id=e9bg9o6hp6nr5cdb4j4r]
yandex_storage_object.backend: Refreshing state... [id=terraform.tfstate]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # yandex_iam_service_account.service will be destroyed
  - resource "yandex_iam_service_account" "service" {
      - created_at  = "2025-02-11T14:30:43Z" -> null
      - description = "service account to manage VMs" -> null
      - folder_id   = "b1gfurnb189rkavk8tlr" -> null
      - id          = "aje9pcua6mbl3p4o44pk" -> null
      - name        = "savilovvv" -> null
    }

  # yandex_iam_service_account_static_access_key.terraform_service_account_key will be destroyed
  - resource "yandex_iam_service_account_static_access_key" "terraform_service_account_key" {
      - access_key         = "YCAJEDdt9S2decbO3XI0VhTXk" -> null
      - created_at         = "2025-02-11T14:30:45Z" -> null
      - description        = "static access key for object storage" -> null
      - id                 = "ajeevp64o5h0f8t1c9c3" -> null
      - secret_key         = (sensitive value) -> null
      - service_account_id = "aje9pcua6mbl3p4o44pk" -> null
    }

  # yandex_resourcemanager_folder_iam_member.editor will be destroyed
  - resource "yandex_resourcemanager_folder_iam_member" "editor" {
      - folder_id = "b1gfurnb189rkavk8tlr" -> null
      - id        = "b1gfurnb189rkavk8tlr/editor/serviceAccount:aje9pcua6mbl3p4o44pk" -> null
      - member    = "serviceAccount:aje9pcua6mbl3p4o44pk" -> null
      - role      = "editor" -> null
    }

  # yandex_storage_bucket.state_storage will be destroyed
  - resource "yandex_storage_bucket" "state_storage" {
      - access_key            = "YCAJEDdt9S2decbO3XI0VhTXk" -> null
      - bucket                = "state-storage1-11-02-2025" -> null
      - bucket_domain_name    = "state-storage1-11-02-2025.storage.yandexcloud.net" -> null
      - default_storage_class = "STANDARD" -> null
      - folder_id             = "b1gfurnb189rkavk8tlr" -> null
      - force_destroy         = false -> null
      - id                    = "state-storage1-11-02-2025" -> null
      - max_size              = 0 -> null
      - secret_key            = (sensitive value) -> null
      - tags                  = {} -> null
        # (1 unchanged attribute hidden)

      - anonymous_access_flags {
          - config_read = false -> null
          - list        = false -> null
          - read        = false -> null
        }

      - versioning {
          - enabled = false -> null
        }
    }

  # yandex_storage_object.backend will be destroyed
  - resource "yandex_storage_object" "backend" {
      - access_key   = "YCAJEDdt9S2decbO3XI0VhTXk" -> null
      - acl          = "private" -> null
      - bucket       = "state-storage1-11-02-2025" -> null
      - content_type = "application/octet-stream" -> null
      - id           = "terraform.tfstate" -> null
      - key          = "terraform.tfstate" -> null
      - secret_key   = (sensitive value) -> null
      - source       = "./terraform.tfstate" -> null
      - tags         = {} -> null
    }

  # yandex_vpc_network.vpc0 will be destroyed
  - resource "yandex_vpc_network" "vpc0" {
      - created_at                = "2025-02-11T14:30:43Z" -> null
      - default_security_group_id = "enp1jdgaqlh7si3efo1u" -> null
      - folder_id                 = "b1gfurnb189rkavk8tlr" -> null
      - id                        = "enpifgf362inb6b5mf0j" -> null
      - labels                    = {} -> null
      - name                      = "vpc0" -> null
      - subnet_ids                = [
          - "e2l2746r0p7qrq8v81bh",
          - "e9bg9o6hp6nr5cdb4j4r",
          - "fl8fvmss0rv12jr1siqb",
        ] -> null
        # (1 unchanged attribute hidden)
    }

  # yandex_vpc_subnet.subnet-a will be destroyed
  - resource "yandex_vpc_subnet" "subnet-a" {
      - created_at     = "2025-02-11T14:30:46Z" -> null
      - folder_id      = "b1gfurnb189rkavk8tlr" -> null
      - id             = "e9bg9o6hp6nr5cdb4j4r" -> null
      - labels         = {} -> null
      - name           = "subnet-a" -> null
      - network_id     = "enpifgf362inb6b5mf0j" -> null
      - v4_cidr_blocks = [
          - "10.0.1.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-a" -> null
        # (2 unchanged attributes hidden)
    }

  # yandex_vpc_subnet.subnet-b will be destroyed
  - resource "yandex_vpc_subnet" "subnet-b" {
      - created_at     = "2025-02-11T14:30:46Z" -> null
      - folder_id      = "b1gfurnb189rkavk8tlr" -> null
      - id             = "e2l2746r0p7qrq8v81bh" -> null
      - labels         = {} -> null
      - name           = "subnet-b" -> null
      - network_id     = "enpifgf362inb6b5mf0j" -> null
      - v4_cidr_blocks = [
          - "10.0.2.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-b" -> null
        # (2 unchanged attributes hidden)
    }

  # yandex_vpc_subnet.subnet-d will be destroyed
  - resource "yandex_vpc_subnet" "subnet-d" {
      - created_at     = "2025-02-11T14:30:45Z" -> null
      - folder_id      = "b1gfurnb189rkavk8tlr" -> null
      - id             = "fl8fvmss0rv12jr1siqb" -> null
      - labels         = {} -> null
      - name           = "subnet-d" -> null
      - network_id     = "enpifgf362inb6b5mf0j" -> null
      - v4_cidr_blocks = [
          - "10.0.3.0/24",
        ] -> null
      - v6_cidr_blocks = [] -> null
      - zone           = "ru-central1-d" -> null
        # (2 unchanged attributes hidden)
    }

Plan: 0 to add, 0 to change, 9 to destroy.

Do you really want to destroy all resources?
  Terraform will destroy all your managed infrastructure, as shown above.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes

yandex_resourcemanager_folder_iam_member.editor: Destroying... [id=b1gfurnb189rkavk8tlr/editor/serviceAccount:aje9pcua6mbl3p4o44pk]
yandex_vpc_subnet.subnet-b: Destroying... [id=e2l2746r0p7qrq8v81bh]
yandex_storage_object.backend: Destroying... [id=terraform.tfstate]
yandex_vpc_subnet.subnet-a: Destroying... [id=e9bg9o6hp6nr5cdb4j4r]
yandex_vpc_subnet.subnet-d: Destroying... [id=fl8fvmss0rv12jr1siqb]
yandex_storage_object.backend: Destruction complete after 0s
yandex_storage_bucket.state_storage: Destroying... [id=state-storage1-11-02-2025]
yandex_vpc_subnet.subnet-d: Destruction complete after 0s
yandex_vpc_subnet.subnet-a: Destruction complete after 1s
yandex_vpc_subnet.subnet-b: Destruction complete after 1s
yandex_vpc_network.vpc0: Destroying... [id=enpifgf362inb6b5mf0j]
yandex_vpc_network.vpc0: Destruction complete after 1s
yandex_resourcemanager_folder_iam_member.editor: Destruction complete after 3s
yandex_storage_bucket.state_storage: Still destroying... [id=state-storage1-11-02-2025, 10s elapsed]
yandex_storage_bucket.state_storage: Destruction complete after 12s
yandex_iam_service_account_static_access_key.terraform_service_account_key: Destroying... [id=ajeevp64o5h0f8t1c9c3]
yandex_iam_service_account_static_access_key.terraform_service_account_key: Destruction complete after 0s
yandex_iam_service_account.service: Destroying... [id=aje9pcua6mbl3p4o44pk]
yandex_iam_service_account.service: Destruction complete after 3s

Destroy complete! Resources: 9 destroyed.
```
