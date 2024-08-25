### Useful commands

## Management Commands
```shell
# Create a trends request
docker-compose exec web python manage.py post_trending
# Check to see if the request was fulfilled.
# If so, fetch daily trends
docker-compose exec web python manage.py get_trending

```

```shell
#### BASE COMMANDS
# Start local docker instances
docker-compose up -d --build
# Take instances down
docker-compose down --remove-orphans
# View logs
docker logs --follow recipe_research-web-1
# Run tests
docker-compose exec web python manage.py test

# Django migrations
docker-compose exec web python manage.py makemigrations
# ONLY RUN LOCALLY
docker-compose exec web python manage.py migrate

# Connect to Postgres
docker-compose exec db psql --username=postgres --dbname=postgres
# run your SQL commands now

# Wipe DB
sudo docker volume rm recipe_research_postgres_data

```

### Postgres DB backup/restore

```shell
docker exec b07c0d3d66e1 pg_dump -U postgres >backup.sql
# Backup specific tables
docker exec b07c0d3d66e1 pg_dump -U postgres -d postgres -t watchmyteam_arena -t watchmyteam_league -t watchmyteam_season -t watchmyteam_team -t watchmyteam_game -t
watchmyteam_distance >watchmyteam.sql

# restore backup
# copy file to container
sudo docker cp backup.sql edd255f87e94:/
# load backup file
sudo docker exec 5ca62fdcdefe psql -U postgres -d postgres -f /backup.sql

```

### Delete today's Trending KW

```postgresql
DELETE
FROM trends_trending
WHERE DATE(created_on) = CURRENT_DATE;
```
### Manually insert a row

```postgresql
--- Do a SELECT to verify columns
SELECT *
FROM trends_trendingbatch
LIMIT 1;

--- INSERT SQL
INSERT INTO trends_trendingbatch
VALUES (1, '07210409-4355-0170-0000-fb6660100e70', 'google.com', 't', NOW());
```
### Deleting duplicate keywords

```postgresql
--  Delete keyword from Batch Result
DELETE
FROM trends_batchresult
WHERE keyword_id IN (SELECT a.kw_id
                     FROM trends_keyword a
                              JOIN trends_keyword b ON a.keyword = b.keyword AND a.kw_id < b.kw_id);

--  Delete keyword from Trending
DELETE
FROM trends_trending
WHERE keyword_id IN (SELECT a.kw_id
                     FROM trends_keyword a
                              JOIN trends_keyword b ON a.keyword = b.keyword AND a.kw_id < b.kw_id);

--  Delete keyword from RankingForumPosts
DELETE
FROM trends_rankingforumposts
WHERE keyword_id IN (SELECT a.kw_id
                     FROM trends_keyword a
                              JOIN trends_keyword b ON a.keyword = b.keyword AND a.kw_id < b.kw_id);


--  Delete keyword from Related Keyword 
DELETE
FROM trends_relatedkeyword
WHERE related_keyword_id IN (SELECT a.kw_id
                             FROM trends_keyword a
                                      JOIN trends_keyword b ON a.keyword = b.keyword AND a.kw_id < b.kw_id);
DELETE
FROM trends_relatedkeyword
WHERE keyword_id IN (SELECT a.kw_id
                     FROM trends_keyword a
                              JOIN trends_keyword b ON a.keyword = b.keyword AND a.kw_id < b.kw_id);

--  Delete the keyword
delete
from trends_keyword a using trends_keyword b
where a.kw_id < b.kw_id
  AND a.keyword = b.keyword;

-- Verify
select a.kw_id
from trends_keyword AS a,
     trends_keyword AS b
where a.kw_id < b.kw_id
  AND a.keyword = b.keyword;


```

### AWS

```shell
# copy file to aws
scp -i ~/Documents/Development/AWS/2022RTW.pem watchmyteam.sql ubuntu@34.229.15.11:~/

```