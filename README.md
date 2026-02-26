!/bin/bash

ORACLE_HOME=/u01/app/oracle/product/19/dbee_Q4
ORACLE_SID=P05
PATH=$ORACLE_HOME/bin:$PATH

THRESHOLD=85

RESULT=$(sqlplus -s / as sysdba <<EOF
SET HEAD OFF FEED OFF
SELECT ROUND((SUM(pga_used_mem)/
       (SELECT value FROM v\\$parameter WHERE name='pga_aggregate_target')
       )*100,2)
FROM v\\$process;
EXIT;
EOF
)

PGA_USAGE=$(echo $RESULT | xargs)

if (( $(echo "$PGA_USAGE > $THRESHOLD" | bc -l) )); then
   echo "ALERT: PGA usage is $PGA_USAGE%" | mail -s "Oracle PGA Alert" ab@ae.com
fi.
