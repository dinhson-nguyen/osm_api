# iotlink_osm_api
2023_iotlink_osm_api

First, you need to install Overpass api . description here http://overpass-api.de/full_installation.html
Note that there is cache in log folder will increase unlimit. setting file launch.sh not to create them. 
demo file launch.sh below 


h#!/usr/bin/env bash

EXEC_DIR="/opt/op/bin"
DB_DIR="/opt/op/db"
DIFF_DIR="/opt/op/diff"
LOG_DIR="/opt/op/log"

rm -fv $DB_DIR/osm3s_v0.7.5*
rm -fv $DB_DIR/*.shadow
rm -fv /dev/shm/osm3s*

ionice -c 2 -n 7 nice -n 17 nohup \
    "$EXEC_DIR/dispatcher" --osm-base --attic --rate-limit=2 \
    --space=10737418240 "--db-dir=$DB_DIR" >>"$LOG_DIR/osm_base.out" &
ionice -c 2 -n 7 nice -n 18 nohup \
    "$EXEC_DIR/dispatcher" --areas "--db-dir=$DB_DIR" >>"$LOG_DIR/areas.out" &

   
ionice -c 3 nice -n 17 nohup \
    "$EXEC_DIR/apply_osc_to_db.sh" "$DIFF_DIR" auto --meta=attic \
    >>"$LOG_DIR/apply_osc_to_db.out" &

"$EXEC_DIR/area_updater.sh" &


 -- and area_updater.sh below:


 
 DB_DIR="/opt/op/db"
EXEC_DIR="/opt/op/bin"
LOG_DIR="/opt/op/log"

pushd "$EXEC_DIR"

while [[ true ]]; do
{
  echo "`date '+%F %T'`: update started" >>$LOG_DIR/area_update.log
  ionice -c 2 -n 7 nice -n 19 "$EXEC_DIR/osm3s_query" --progress \
    --rules <$DB_DIR/rules/areas.osm3s >>$LOG_DIR/area_update.log
  echo "`date '+%F %T'`: update finished" >>$LOG_DIR/area_update.log
  sleep 3
}; done




