# Module 1 Homework: Docker & SQL

In this homework we'll prepare the environment and practice
Docker and SQL

When submitting your homework, you will also need to include
a link to your GitHub repository or other public code-hosting
site.

This repository should contain the code for solving the homework. 

When your solution has SQL or shell commands and not code
(e.g. python files) file formad, include them directly in
the README file of your repository.


## Question 1. Understanding docker first run 

Run docker with the `python:3.12.8` image in an interactive mode, use the entrypoint `bash`.

What's the version of `pip` in the image?

- 24.3.1 --[Selected]
- 24.2.1
- 23.3.1
- 23.2.1
[A1]:
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ sudo docker run -it --entrypoint bash python:3.12.8
Unable to find image 'python:3.12.8' locally
3.12.8: Pulling from library/python
fd0410a2d1ae: Pull complete 
bf571be90f05: Pull complete 
684a51896c82: Pull complete 
fbf93b646d6b: Pull complete 
5f16749b32ba: Pull complete 
e00350058e07: Pull complete 
eb52a57aa542: Pull complete 
Digest: sha256:5893362478144406ee0771bd9c38081a185077fb317ba71d01b7567678a89708
Status: Downloaded newer image for python:3.12.8
root@4e92b52e72ac:/# pip --version
pip 24.3.1 from /usr/local/lib/python3.12/site-packages/pip (python 3.12)
root@4e92b52e72ac:/# 


## Question 2. Understanding Docker networking and docker-compose

Given the following `docker-compose.yaml`, what is the `hostname` and `port` that **pgadmin** should use to connect to the postgres database?

[A2]:Host name : db  and port for posgresqlConnection:5432
services:
  db:
    ports:
      - '5433:5432'
z
```yaml
services:
  db:
    container_name: postgres
    image: postgres:17-alpine
    environment:
      POSTGRES_USER: 'postgres'
      POSTGRES_PASSWORD: 'postgres'
      POSTGRES_DB: 'ny_taxi'
    ports:
      - '5433:5432'
    volumes:
      - vol-pgdata:/var/lib/postgresql/data

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4:latest
    environment:
      PGADMIN_DEFAULT_EMAIL: "pgadmin@pgadmin.com"
      PGADMIN_DEFAULT_PASSWORD: "pgadmin"
    ports:
      - "8080:80"
    volumes:
      - vol-pgadmin_data:/var/lib/pgadmin  

volumes:
  vol-pgdata:
    name: vol-pgdata
  vol-pgadmin_data:
    name: vol-pgadmin_data
```

- postgres:5433
- localhost:5432
- db:5433
- postgres:5432
- db:5432


##  Prepare Postgres

Run Postgres and load data as shown in the videos
We'll use the green taxi trips from October 2019:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/green/green_tripdata_2019-10.csv.gz
```

You will also need the dataset with zones:

```bash
wget https://github.com/DataTalksClub/nyc-tlc-data/releases/download/misc/taxi_zone_lookup.csv
```

Download this data and put it into Postgres.

You can use the code from the course. It's up to you whether
you want to use Jupyter or a python script.

## Question 3. Trip Segmentation Count

During the period of October 1st 2019 (inclusive) and November 1st 2019 (exclusive), how many trips, **respectively**, happened:
1. Up to 1 mile
2. In between 1 (exclusive) and 3 miles (inclusive),
3. In between 3 (exclusive) and 7 miles (inclusive),
4. In between 7 (exclusive) and 10 miles (inclusive),
5. Over 10 miles 

Answers:

- 104,802;  197,670;  110,612;  27,831;  35,281
- 104,802;  198,924;  109,603;  27,678;  35,189
- 104,793;  201,407;  110,612;  27,831;  35,281
- 104,793;  202,661;  109,603;  27,678;  35,189
- 104,838;  199,013;  109,645;  27,688;  35,202 [SELECTED]
[A3]:
SELECT 
  CASE 
    WHEN trip_distance <= 1 THEN '0-1'
    WHEN trip_distance > 1 AND trip_distance <= 3 THEN '1-3'
    WHEN trip_distance > 3 AND trip_distance <= 7 THEN '3-7'
    WHEN trip_distance > 7 AND trip_distance <= 10 THEN '7-10'
    ELSE 'over 10'
  END AS distance_group,
  COUNT(*) as count
FROM green_taxi
WHERE lpep_pickup_datetime >= '2019-10-01' 
  AND lpep_pickup_datetime < '2019-11-01'
GROUP BY 1
ORDER BY 1;

+----------------+--------+
| distance_group | count  |
|----------------+--------|
| 0-1            | 104830 |
| 1-3            | 198995 |
| 3-7            | 109642 |
| 7-10           | 27686  |
| over 10        | 35201  |
+----------------+--------+


## Question 4. Longest trip for each day

Which was the pick up day with the longest trip distance?
Use the pick up time for your calculations.

Tip: For every day, we only care about one single trip with the longest distance. 

- 2019-10-11
- 2019-10-24
- 2019-10-26
- 2019-10-31 [SELECTED]

root@localhost:ny_taxi> SELECT DATE(lpep_pickup_datetime) as pickup_day, MAX(trip_distance) as max_distance
 FROM green_taxi
 WHERE EXTRACT(MONTH FROM lpep_pickup_datetime) = 10
   AND EXTRACT(YEAR FROM lpep_pickup_datetime) = 2019
 GROUP BY 1
 ORDER BY max_distance DESC
 LIMIT 1;
+------------+--------------+
| pickup_day | max_distance |
|------------+--------------|
| 2019-10-31 | 515.89       |
+------------+--------------+
SELECT 1
Time: 0.156s
root@localhost:ny_taxi>

## Question 5. Three biggest pickup zones

Which were the top pickup locations with over 13,000 in
`total_amount` (across all trips) for 2019-10-18?

Consider only `lpep_pickup_datetime` when filtering by date.
 
- East Harlem North, East Harlem South, Morningside Heights
- East Harlem North, Morningside Heights
- Morningside Heights, Astoria Park, East Harlem South
- Bedford, East Harlem North, Astoria Park

[A5]:
root@localhost:ny_taxi> SELECT zl."Zone", SUM(g.total_amount) as total
 FROM green_taxi g
 JOIN taxi_zones zl ON g."PULocationID" = zl."LocationID"
 WHERE DATE(g.lpep_pickup_datetime) = '2019-10-18'
 GROUP BY 1
 HAVING SUM(g.total_amount) > 13000
 ORDER BY total DESC;
+-----------------------------+----------+
| Zone                        | total    |
|-----------------------------+----------|
| East Harlem North           | 74746.72 |
| East Harlem South           | 67189.04 |
| Morningside Heights         | 52119.16 |
| Central Harlem              | 49762.64 |
| Elmhurst                    | 49727.84 |
| Astoria                     | 36763.44 |
| Washington Heights South    | 33796.28 |
| Central Harlem North        | 31962.72 |
| Forest Hills                | 31476.24 |
| Brooklyn Heights            | 30246.52 |
| Fort Greene                 | 29601.24 |
| Jamaica                     | 28503.76 |
| Downtown Brooklyn/MetroTech | 26518.48 |
| Jackson Heights             | 24783.48 |
| DUMBO/Vinegar Hill          | 22408.40 |
| West Concourse              | 18638.76 |
| Woodside                    | 18610.40 |
| Boerum Hill                 | 18410.80 |
| Park Slope                  | 18394.24 |
| Red Hook                    | 17534.88 |
| Hamilton Heights            | 15106.68 |
| East New York               | 14924.40 |
| Steinway                    | 13731.04 |
| Crown Heights North         | 13063.60 |
+-----------------------------+----------+
SELECT 24
Time: 0.095s
root@localhost:ny_taxi>

## Question 6. Largest tip

For the passengers picked up in Ocrober 2019 in the zone
name "East Harlem North" which was the drop off zone that had
the largest tip?

Note: it's `tip` , not `trip`

We need the name of the zone, not the ID.

- Yorkville West
- JFK Airport
- East Harlem North
- East Harlem South

[A6]:
Home: http://pgcli.com
root@localhost:ny_taxi> SELECT dzones."Zone" as dropoff_zone, MAX(g.tip_amount) as max_tip
 FROM green_taxi g
 JOIN taxi_zones pzones ON g."PULocationID" = pzones."LocationID"
 JOIN taxi_zones dzones ON g."DOLocationID" = dzones."LocationID"
 WHERE pzones."Zone" = 'East Harlem North'
 AND DATE(g.lpep_pickup_datetime) >= '2019-10-01'
 AND DATE(g.lpep_pickup_datetime) < '2019-11-01'
 GROUP BY dzones."Zone"
 ORDER BY max_tip DESC
 LIMIT 1;
+--------------+---------+
| dropoff_zone | max_tip |
|--------------+---------|
| JFK Airport  | 87.3    |
+--------------+---------+
SELECT 1
Time: 0.441s
root@localhost:ny_taxi>



## Terraform

In this section homework we'll prepare the environment by creating resources in GCP with Terraform.

In your VM on GCP/Laptop/GitHub Codespace install Terraform. 
Copy the files from the course repo
[here](../../../01-docker-terraform/1_terraform_gcp/terraform) to your VM/Laptop/GitHub Codespace.

Modify the files as necessary to create a GCP Bucket and Big Query Dataset.


## Question 7. Terraform Workflow

Which of the following sequences, **respectively**, describes the workflow for: 
1. Downloading the provider plugins and setting up backend,
2. Generating proposed changes and auto-executing the plan
3. Remove all resources managed by terraform`

Answers:
- terraform import, terraform apply -y, terraform destroy
- teraform init, terraform plan -auto-apply, terraform rm
- terraform init, terraform run -auto-aprove, terraform destroy
- terraform init, terraform apply -auto-aprove, terraform destroy
- terraform import, terraform apply -y, terraform rm

[A7]:
terraform init, terraform apply -auto-approve, terraform destroy

(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ terraform init
Initializing the backend...
Initializing provider plugins...
- Reusing previous version of hashicorp/google from the dependency lock file
- Using previously-installed hashicorp/google v5.12.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ terraform apply -auto-approve

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # google_bigquery_dataset.dataset will be created
  + resource "google_bigquery_dataset" "dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "trips_data_all"
      + default_collation          = (known after apply)
      + delete_contents_on_destroy = false
      + effective_labels           = (known after apply)
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + is_case_insensitive        = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "US"
      + max_time_travel_hours      = (known after apply)
      + project                    = "gino-viloria-dtc-de-bootcamp"
      + self_link                  = (known after apply)
      + storage_billing_model      = (known after apply)
      + terraform_labels           = (known after apply)

      + access (known after apply)
    }

  # google_storage_bucket.data-lake-bucket will be created
  + resource "google_storage_bucket" "data-lake-bucket" {
      + effective_labels            = (known after apply)
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "US"
      + name                        = "dtc-data-lake-bucket-ginovilo"
      + project                     = (known after apply)
      + public_access_prevention    = (known after apply)
      + rpo                         = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + terraform_labels            = (known after apply)
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type          = "Delete"
                # (1 unchanged attribute hidden)
            }
          + condition {
              + age                    = 30
              + matches_prefix         = []
              + matches_storage_class  = []
              + matches_suffix         = []
              + with_state             = (known after apply)
                # (3 unchanged attributes hidden)
            }
        }

      + versioning {
          + enabled = true
        }

      + website (known after apply)
    }

Plan: 2 to add, 0 to change, 0 to destroy.
google_bigquery_dataset.dataset: Creating...
google_storage_bucket.data-lake-bucket: Creating...
google_bigquery_dataset.dataset: Creation complete after 1s [id=projects/gino-viloria-dtc-de-bootcamp/datasets/trips_data_all]
google_storage_bucket.data-lake-bucket: Creation complete after 2s [id=dtc-data-lake-bucket-ginovilo]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ gsutil ls
gs://dtc-data-lake-bucket-ginovilo/
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ bq ls
    datasetId     
 ---------------- 
  trips_data_all  
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ terraform destroy -auto-approve
google_storage_bucket.data-lake-bucket: Refreshing state... [id=dtc-data-lake-bucket-ginovilo]
google_bigquery_dataset.dataset: Refreshing state... [id=projects/gino-viloria-dtc-de-bootcamp/datasets/trips_data_all]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # google_bigquery_dataset.dataset will be destroyed
  - resource "google_bigquery_dataset" "dataset" {
      - creation_time                   = 1736998104604 -> null
      - dataset_id                      = "trips_data_all" -> null
      - default_partition_expiration_ms = 0 -> null
      - default_table_expiration_ms     = 0 -> null
      - delete_contents_on_destroy      = false -> null
      - effective_labels                = {} -> null
      - etag                            = "eck7h8V+Bq7ADhjKRtg0wQ==" -> null
      - id                              = "projects/gino-viloria-dtc-de-bootcamp/datasets/trips_data_all" -> null
      - is_case_insensitive             = false -> null
      - labels                          = {} -> null
      - last_modified_time              = 1736998104604 -> null
      - location                        = "US" -> null
      - max_time_travel_hours           = "168" -> null
      - project                         = "gino-viloria-dtc-de-bootcamp" -> null
      - self_link                       = "https://bigquery.googleapis.com/bigquery/v2/projects/gino-viloria-dtc-de-bootcamp/datasets/trips_data_all" -> null
      - terraform_labels                = {} -> null
        # (4 unchanged attributes hidden)

      - access {
          - role           = "OWNER" -> null
          - user_by_email  = "terraform-runner-ginovilo@gino-viloria-dtc-de-bootcamp.iam.gserviceaccount.com" -> null
            # (4 unchanged attributes hidden)
        }
      - access {
          - role           = "OWNER" -> null
          - special_group  = "projectOwners" -> null
            # (4 unchanged attributes hidden)
        }
      - access {
          - role           = "READER" -> null
          - special_group  = "projectReaders" -> null
            # (4 unchanged attributes hidden)
        }
      - access {
          - role           = "WRITER" -> null
          - special_group  = "projectWriters" -> null
            # (4 unchanged attributes hidden)
        }
    }

  # google_storage_bucket.data-lake-bucket will be destroyed
  - resource "google_storage_bucket" "data-lake-bucket" {
      - default_event_based_hold    = false -> null
      - effective_labels            = {} -> null
      - enable_object_retention     = false -> null
      - force_destroy               = true -> null
      - id                          = "dtc-data-lake-bucket-ginovilo" -> null
      - labels                      = {} -> null
      - location                    = "US" -> null
      - name                        = "dtc-data-lake-bucket-ginovilo" -> null
      - project                     = "gino-viloria-dtc-de-bootcamp" -> null
      - public_access_prevention    = "inherited" -> null
      - requester_pays              = false -> null
      - rpo                         = "DEFAULT" -> null
      - self_link                   = "https://www.googleapis.com/storage/v1/b/dtc-data-lake-bucket-ginovilo" -> null
      - storage_class               = "STANDARD" -> null
      - terraform_labels            = {} -> null
      - uniform_bucket_level_access = true -> null
      - url                         = "gs://dtc-data-lake-bucket-ginovilo" -> null

      - lifecycle_rule {
          - action {
              - type          = "Delete" -> null
                # (1 unchanged attribute hidden)
            }
          - condition {
              - age                        = 30 -> null
              - days_since_custom_time     = 0 -> null
              - days_since_noncurrent_time = 0 -> null
              - matches_prefix             = [] -> null
              - matches_storage_class      = [] -> null
              - matches_suffix             = [] -> null
              - no_age                     = false -> null
              - num_newer_versions         = 0 -> null
              - with_state                 = "ANY" -> null
                # (3 unchanged attributes hidden)
            }
        }

      - versioning {
          - enabled = true -> null
        }
    }

Plan: 0 to add, 0 to change, 2 to destroy.
google_storage_bucket.data-lake-bucket: Destroying... [id=dtc-data-lake-bucket-ginovilo]
google_bigquery_dataset.dataset: Destroying... [id=projects/gino-viloria-dtc-de-bootcamp/datasets/trips_data_all]
google_bigquery_dataset.dataset: Destruction complete after 1s
google_storage_bucket.data-lake-bucket: Destruction complete after 1s

Destroy complete! Resources: 2 destroyed.
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ gsutil ls
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ bq ls
(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ conda env list
# conda environments:
#
base                  *  /home/gv-ia/anaconda3
dtc-data-bootcamp-gino     /home/gv-ia/anaconda3/envs/dtc-data-bootcamp-gino

(base) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ conda activate dtc-data-bootcamp-gino
(dtc-data-bootcamp-gino) gv-ia@gv-ia:~/data-engineering-zoomcamp/test/terraform$ 


## Submitting the solutions

* Form for submitting: https://courses.datatalks.club/de-zoomcamp-2025/homework/hw1
