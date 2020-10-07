task13

```
#!/bin/bash

if [ ${#} -ne 1 ]; then
        echo "The number of arguments is incorrect!"
        exit 1
fi

if [ ! -d "${1}" ]; then
        echo "The given argument is not a directory!"
        exit 2
fi

find "${1}" -type l ! -exec test -e '{}' \; -printf "%p\n" 2>/dev/null
```
----------------------------------------------

```
#!/bin/bash

if [ "$#" -ne 1 ] ; then
        echo "1"
        exit 1
fi
if [ ! -d "$1" ] ; then
        echo "2"
        exit 2
fi
find "$1" -type l 2>/dev/null | while read line
do
        if [ ! -e "${line}" ] ; then
                echo "${line}"
        fi
done
```

task14

```
#!/bin/bash

if [ ${#} -ne 1 ]; then
	echo "Invalid number of arguments!"
	exit 1
fi

echo "${1}" | egrep "^[+-]?[0-9]+$" &>/dev/null
if [ "${?}" -ne 0 ]; then 
	echo "The given argument is not an integer!"
	exit 2
fi

if [ $(id -u) -ne 0 ]; then
	echo "It is not root!"
	exit 3
fi	

for line in $(ps -e -o uid | tail -n +2 | sort -n | uniq ); do
	TEMP=$(ps -u $line -o uid,rss | tail -n +2 | awk 'BEGIN { cnt=0 } { cnt+=$2 } END { printf("%s %s\n",$1,cnt) }')
	echo $TEMP
	sum=$(echo $TEMP | awk '{ print $2 }')
	if [ ${sum} -gt ${1} ]; then
	max=$(ps -u $line -o pid,rss | tail -n +2 | sort -n -k 2| tail -n 1 | awk '{ print $1 }')
	kill -SIGTERM ${max} 2>/dev/null
	sleep 1 
	kill -SIGKILL ${max} 2>/dev/null
	fi
	 
done
```
-----------------------------------------------------

```
#!/bin/bash

if [ $(id -u) -ne 0 ] ; then
        echo "1"
        exit 1
fi
if [ "$#" -ne 1 ] ; then
        echo "2"
        exit 2
fi
if ! echo "$1" | egrep "^[+-]?[0-9]+$" >/dev/null ; then
       echo "3"
       exit 3
fi
for user in $(ps -e -o uid | tail -n +2 | sort -u); do
        sum=$(ps -u $user -o uid,rss | awk 'BEGIN {sum=0} {sum+=$2} END {print sum}')
        echo "$user $sum"
        if [ "${sum}" -gt "$1" ] ; then
                pid=$(ps -u $user -o pid,rss | tail -n +2 | sort -k 2| tail -n 1 | awk '{print $1}')
                kill -SIGTERM $pid 2>/dev/null
                sleep 1
                kill -SIGKILL $pid 2>/dev/null

        fi
done
```

----------------------------------------------------- 

pdf15

```
#!/bin/bash

if [ $(id -u) -ne 0 ]; then
	echo "It is not root!"
	exit 1
fi

for line in $(cat /etc/passwd | cut -d ':' -f 1,6) ; do
	USER=$(echo "${line}" | awk -F ":" '{ print $1 }')
	HOMEDIR=$(echo "${line}" | awk -F ":" '{ print $2 }')
	if sudo -u "${USER}" [ -z "${HOMEDIR}" -o ! -d "${HOMEDIR}" -o ! -w "${HOMEDIR}" ] 2>/dev/null; then
	       echo "${USER} homedir doesn't exist or cannot write there!"
        fi	       
done

exit 0
```
----------------------------------------------------
pdf 16

```
#!/bin/bash

if [ ${#} -ne 3 ]; then
	echo "Invalid number of arguments!"
	exit 1
fi

if [ ! -w "${1}" -o ! -r "${1}" ]; then
	echo "exit 2"
	exit 2
fi

cat "${1}" | egrep "^${2}=" &>/dev/null
if [ ${?} -ne 0 ]; then
	echo "The first string doesn't exist!"
	exit 3
fi

cat "${1}" | egrep "^${3}=" &>/dev/null
if [ ${?} -ne 0 ]; then
        exit 0
fi     

STR1=$(cat "${1}" | egrep "^${2}=" | cut -d '=' -f 2- | sed -E 's/[][\.$()|*+?{}^]/\\&/g')
STR2=$(cat "${1}" | egrep "^${3}=" | cut -d '=' -f 2- | sed -E 's/[][\.$()|*+?{}^]/\\&/g')
#neekumreshenieto
STR3=$(cat "${1}" | egrep "^${3}=" | cut -d '=' -f 2- | sed -E 's/[][\.$()|*+?{}^]/\\&/g' | tr ' ' '\n' | sort | uniq -d | tr '\n' ' ')

for rep in ${STR3}; do
	for line in ${STR2}; do
	if [ $rep = $line ]; then
		if echo "${STR2}" | egrep "$rep$" &>/dev/null; then
                STR2=$(echo "${STR2}" | sed -E "s/ $rep//")
        	fi
        	if echo "${STR2}" | egrep "$rep " &>/dev/null; then
                STR2=$(echo "${STR2}" | sed -E "s/$rep //")
        	fi
	fi
	done
done

for line1 in ${STR1}; do
	for line2 in ${STR2}; do
	if [ $line1 = $line2 ]; then
	if echo "${STR2}" | egrep "$line1$" &>/dev/null; then
		STR2=$(echo "${STR2}" | sed -E "s/ $line1//g")
	fi
	if echo "${STR2}" | egrep "$line1 " &>/dev/null; then
		STR2=$(echo "${STR2}" | sed -E "s/$line1 //g")
        fi
	fi
	done
done

echo "${STR2}"

sed -E -i "s/$3=.*$/$3=$STR2/" "$1"
```

---------------------------------------------------
17pdf

```
#!/bin/bash

if [ $# -ne 1 ]; then
	echo "Invalid number of arguments!"
	exit 1
fi

if [ $(id -u) -ne 0 ]; then
	echo "It is not root!"
	exit 2
fi

CNT=$(ps -u "${1}" -o user= | wc -l)

for line in $(ps -e -o user= | sort | uniq); do
	if [ "${line}" != "${1}" ]; then
		if [ $CNT -lt $(ps -u "${line}" | wc -l) ]; then
	       		echo "$line"
        	fi
	fi
done

NUM=0
TIME=0
for i in $(ps -e -o time= | awk -F ':' '{ print (($1 * 3600) + ($2 * 60) + $3) }')
do
	TIME=$(($TIME + $i))
	NUM=$(($NUM + 1))
done

SUM=$(($TIME/$NUM))
echo "$SUM"

TIMES=$(($SUM * 2))
ps -u ${1} -o time= -o pid= | while read i
do
	SEC=$(echo "$i" | awk '{ print $1 }' | awk -F ':' '{ print (($1 * 3600) + ($2 * 60) + $3) }')
	PID=$( echo "$i" | awk '{ print $2 }')
        if [ $SEC -gt $TIMES ]; then
		kill -TERM $PID 2>/dev/null
	        sleep 2
		kill -KILL $PID 2>/dev/null
	fi

done

exit 0
```

-----------------------------------------------------------------

18pdf

```
#!/bin/bash

cat /etc/passwd | cut -d ':' -f 1,6 | while read line
do
        SEC=$(echo "${line}" | cut -d ':' -f 2)
        FIRST=$(echo "${line}" | cut -d ':' -f 1)
        MMTIME=$(find $SEC -maxdepth 1 -type f -printf '%T@\n' 2>/dev/null | sort -n | tail -n 1)
        find $SEC -maxdepth 1 -type f -printf '%T@ %p\n' 2>/dev/null | awk -v m=$MMTIME -v f=$FIRST '{ if ( $1==m ) { printf("%s %s %s\n",f,$2,$1) } }'
done | sort -n -k 3 | tail -n 1 | cut -d " " -f 1,2
```

--------------------------------------------------------------------------------

```
#!/bin/bash

cut -d ':' -f 1,6 /etc/passwd | \
while read line; do
    dir=$(echo "$line" | cut -d ':' -f 2)
    [ -d "$dir" ] || continue
    find "$dir" -maxdepth 1 -type f -printf "%T@ $(echo "$line" | cut -d ':' -f 1) %p\n" 2> /dev/null
done | sort -n -k 1,1 | tail -n 1 | cut -d ' ' -f 2-
exit 0
```

-------------------------------------------------------------------------------------------

```
#!/bin/bash

cat /etc/passwd | cut -d ':' -f 1,6 | while read line
do
        SEC=$(echo "${line}" | cut -d ':' -f 2)
        FIRST=$(echo "${line}" | cut -d ':' -f 1)
        find $SEC -maxdepth 1 -type f -printf '%T@ %p\n' 2>/dev/null | sort -k 1| tail -n 1 | awk -v f=$FIRST '{ printf("%s %s %s\n",f,$2,$1) }'
done | sort -n -k 3 | tail -n 1 | cut -d " " -f 1,2
```

----------------------------------------------------------------------------

```
#!/bin/bash


cat /etc/passwd | cut -d ':' -f 1,6 | while read line
do
        NAME=$(echo "${line}" | cut -d ':' -f 1)
        DIR=$( echo "${line}" | cut -d ':' -f 2)
        while read l; do
        echo "${l} ${NAME}"
        done < <(find "${DIR}" -maxdepth 1 -type f -printf "%T@ %p\n" 2>/dev/null)
done | sort -t ' ' -k 1,1n | uniq | tail -n 1 | awk '{ printf("%s %s\n",$3,$2) }'
```

-------------------------------------------------------------------------
19 pdf

```
#!/bin/bash

if [ $# -lt 1 -o $# -gt 2 ];then
    echo "Invalid number of arguments!"
    exit 1
fi

if [ ! -d "${1}" ]; then 
    echo "The first argument must be a directory!"
    exit 2
fi

if [ $# -eq 2 ];then
    echo $2 | egrep "^[+-]?[0-9]+$" &>/dev/null
    if [ $? -ne 0 ];then
        echo "The second argument must be an integer!"
        exit 3
    fi

    find "${1}" -type f -printf "%p %n\n" 2>/dev/null | while read line
    do
        HD=$(echo "${line}" | cut -d ' ' -f 2)
        if [ $HD -ge $2 ]; then
            echo "${line}"
        fi
    done
fi

if [ $# -eq 1 ];then
    find "${1}" -type l -exec test ! -e '{}' \; -printf "%p\n" 2>/dev/null
fi
```

------------------------------------------------------------------

```
#!/bin/bash
if [ $# -lt 1 -o $# -gt 2 ]
then
    echo "Invalid number of parameter"
    exit 1
fi
if [ ! -d $1 ]
then 
    echo "Permission denied with directory"
    exit 2
fi
if [ $# -eq 2 ]
then
    if ! echo "$2"|egrep -q "^[0-9]+$"
    then
        echo "Error format for second parameter"
        exit 3
    fi
    l=$(($2-1))
    find -links $l -printf "%p\n" 2>/dev/null
else
    find -type l -printf "%p\n" 2>/dev/null|while read l
    do
        if [ -e $l ]
        then 
            echo "$l"
        fi
    done
fi
```

---------------------------------------------------
20 pdf

```
#!/bin/bash

if [ $# -ne 3 ];then
	echo"Invalid number of argumenst!"
	exit 1
fi

if [ ! -d "${1}" -o ! -d "${2}" ]; then
       echo "The arguments are not directories!"
       exit 2
fi       

if [ $(find "${2}" -maxdepth 1 -mindepth 1 2>/dev/null| wc -l) -ne 0 ];then
	echo "The destination contains files!"
	exit 3
fi
#[ "$(find "$2" -maxdepth 0 ! -empty)" ];
if [ $(id -u) -ne 0 ];then
	echo "permissions denied"
	exit 4
fi

find "${1}" -name "*${3}*" -printf "%P\n" 2>/dev/null | while read line
do
	FILE=$(echo "${line}" | awk -F '/' '{ print $NF }')
	DIRS=$(echo "${line}" | awk -F '/' 'BEGIN {OFS="/";} {$NF=""; print $0;}')
	echo "$f" | grep -E -o "[^/]+$" | fgrep -q "$3" || continue
	mkdir -p ${2}/${DIRS}
	mv ${1}/${line} ${2}/${DIRS}/${FILE}
done
```
---------------------------------------------------------
21 pdf 

```
#!/bin/bash

if [ $(id -u) -ne 0 ];then
	echo "not root"
	exit 1
fi

ps -e -o user= | sort | uniq | while read line
do
	ALL=$(ps -u ${line} -o pid= | wc -l)
	SUM=$(ps -u ${line} -o rss=  | awk '{ SUM+=$1 } END { print SUM}')
	AVG=$(($SUM/$ALL))
	RSS=$(ps -u ${line} -o rss= -o pid= | sort -n | tail -n 1 | awk '{ printf("%s %s\n",$1,$2) }')
	R=$(echo "${RSS}" | cut -d ' ' -f 1)
        PID=$(echo "${RSS}" | cut -d ' ' -f 2)
	echo "${line} $ALL $SUM"
	if [ $(ps -u ${line}) = "root" ]; then
		continue
	fi
	if [ $R -gt "$(($AVG*2))" ]; then
		kill -TERM $PID 2>/dev/null
		sleep 2
		kill -KILL $PID 2>/dev/null
	fi

done
```

---------------------------------------------------------

```
#!/bin/bash

[ $(id -u) -eq 0 ] || { echo "permissions denied" >&2; exit 1; }

res=$(ps -eo user,rss | tail -n +2 | \
 awk '{cnt[$1]++; sum[$1]+=$2;} END {for(i in cnt) printf("%s %s %s\n", i, cnt[i], sum[i]);}')
# a)
echo "$res"
# b)
while read line; do
    [ $(echo "$line" | cut -d ' ' -f 1) = root ] && continue
    pid=$(ps -u $(echo "$line" | cut -d ' ' -f 1) -o rss,pid | tail -n +2 | sort -n -r | head -n 1 | \
            awk -v cnt=$(echo "$line" | cut -d ' ' -f 2) -v sum=$(echo "$line" | cut -d ' ' -f 3) \
             '$1 > 2*sum/cnt {print $2;}')
    kill $pid 2> /dev/null
    sleep 1
    kill -9 $pid 2> /dev/null
done < <(echo "$res")
exit 0
```

------------------------------------------------------------------------------
22 pdf

```
#!/bin/bash

if [ $# -lt 1 -o $# -gt 2 ];then
	echo "Invalid number of arguments!"
	exit 1
fi

if [ ! -d "${1}" ]; then
	echo "The first argument is not a directory!"
	exit 2
fi

if [ $# -eq 2 ]; then
	if [ ! -f "${2}" ]; then
		echo "The sec argument is not a file!"
		exit 3
	fi
fi

CNT=0
if [ $# -eq 2 ];then
	while read line
	do 
		dest=$(echo ${line} | awk '{ print $2 }')
		link=$(echo ${line} | awk '{ print $1 }')
		if [ ! -e ${link} ]; then
			CNT=$(($CNT+1))
		else
		echo "$link -> $dest" 1>>$2
		fi
	done < <(find ${1} -type l -printf "%p %l\n" 2>/dev/null)
	echo "Broken symlinks: $CNT" 1>>$2
else
        while read line
        do
                dest=$(echo ${line} | awk '{ print $2 }')
                link=$(echo ${line} | awk '{ print $1 }')
                if [ ! -e ${link} ]; then
                        CNT=$(($CNT+1))
                else
                echo "$link -> $dest"
                fi
        done < <(find ${1} -type l -printf "%p %l\n" 2>/dev/null)
        echo "Broken symlinks: $CNT"
fi

exit 0
```
----------------------------------------------------------------------------------
23 pdf

```
#!/bin/bash

if [ $# -ne 2 ]; then
    echo "Invalid number of arguments!"
    exit 1
fi

if [ ! -d ${1} ];then
    echo "The second argument is not a directory!"
    exit 2
fi
x1=0
y1=0
z1=0
while read line
do
    x=$(echo ${line} | cut -d '-' -f 2 | cut -d '.' -f 1)
    if [ $x -ge $x1 ]; then
        x1=$x
    fi
done < <(find ${1}  -maxdepth 1 -type f -printf "%f\n" 2>/dev/null | egrep "^vmlinuz-[0-9]+.[0-9]+.[0-9]+-$2$")

while read line
do
        y=$(echo ${line} | cut -d '.' -f 2 )

        if [ $y -ge $y1 ]; then
                y1=$y
        fi
done < <(find ${1}  -maxdepth 1 -type f -printf "%f\n" 2>/dev/null | egrep "^vmlinuz-$x1.[0-9]+.[0-9]+-$2$")

while read line
do
        z=$(echo ${line} | cut -d '.' -f 3 | cut -d '-' -f 1)

        if [ $z -ge $z1 ]; then
                z1=$z
        fi
done < <(find ${1}  -maxdepth 1 -type f -printf "%f\n" 2>/dev/null | egrep "^vmlinuz-$x1.$y1.[0-9]+-$2$")

find ${1}  -maxdepth 1 -type f -printf "%f\n" 2>/dev/null | egrep "^vmlinuz-$x1.$y1.$z1-$2$"
```

--------------------------------------------------------------------
24 pdf 

```
#!/bin/bash

if [ $(id -u) -ne 0 ]; then
	echo "not root"
	exit 1
fi

cat /etc/passwd | awk -F ':' '{ printf("%s %s\n",$1,$6)}' | while read line
do
	name=$(echo ${line} | awk '{ print $1 }')
	if [ $(id -u ${name}) -eq 0 ]; then 
		continue
	fi
	ROOTSIZE=$(ps -u root -o rss= | awk ' BEGIN { CNT=0 } { CNT+=$1 } END { print CNT }')
	dir=$(echo ${line} | awk '{ print $2 }')
	
	if [ -z ${dir} ] || [ ! -d ${dir} ] || [ $(stat --printf="%U\n" ${dir}) != ${name} ] || sudo -u "${name}" [ ! -w ${dir} ]; then 
		CMD=$(ps -u ${name} -o rss=,user=,pid=,uid= | awk '{ printf("%s %s %s %s\n",$1,$2,$3,$4) }')
		RSS=$(echo "${CMD}" | awk ' BEGIN { CNT=0 } { CNT+=$1 } END { print CNT }')
		if [ -n "$CMD" ];then
		echo "${CMD}"
		fi
		if [ $RSS -gt $ROOTSIZE ];then
		killall -TERM -u ${name} 2>/dev/null
		sleep 2
		killall -KILL -u ${name} 2>/dev/null
		fi
	fi
done
```

-------------------------------------------------------------------- s file

```
#!/bin/bash
if [ $(id -u) -ne 0 ]
then
    echo ""
    exit 1
fi
cat /etc/passwd| awk -F ":" '{print $1 " " $6}' |while read l
do
    name=$(echo "$l"|awk '{print $1}')
    dir=$(echo "$l"|awk '{print $2}')
    if [ $(id -u "$name") -eq 0 ]
    then
        continue
    fi
    RootRSS=$(ps -u root -o rss=|awk '{sum+=$1}END{print sum}')
    if [ -z "$dir" ] || [  ! -d "$dir" ] || [ $(stat --printf "%U" "$dir") != "$name" ] || sudo -u "$name" [ ! -w "$dir" ]
    then
        file=$(mktemp)
        trap "rm $file" EXIT
        ps -u "$name" -o user,pid,rss,cmd|tail -n +2 > "$file"
        cat "$file"
        n=$(cat "$file"|awk -v v=$RootRSS '{sum+=$3}END{if(sum>v)printf("%s\n", $1);}')
        killall -15 -u $n 2>/dev/null
        sleep 1
        killall -9 -u $n 2>/dev/null
        #rm "$file"
    fi
done
```

--------------------------------------------------------------------
25 pdf

```
#!/bin/bash

if [ $# -ne 1 ]; then 
	echo "Invalid number of arguments!"
	exit 1
fi

if [ ! -d "${1}" ]; then 
	echo "Invalid first argument!"
	exit 2
fi

{ 
find "${1}" -mindepth 3 -maxdepth 3 -type d -printf "%f 0\n" 2>/dev/null
find "${1}" -mindepth 4 -maxdepth 4 -type f -printf "%f %h\n" 2>/dev/null |while read l
do
	file=$(echo $l | cut -d ' ' -f 1)
	dir=$(echo $l | cut -d ' ' -f 2)
	friend=$( echo $dir | awk -F '/' '{ print $NF }')
	DATE=$(echo "${file}" | egrep "^[0-9]{4}(-[0-9]{2}){5}\.txt$" | cut -d '.' -f 1 | sed -E "s/(.{10})-(.{2})-(.{2})-(.{2})/\1 \2:\3:\4/" )
	date -d "${DATE}" &>/dev/null
	if [ $? -ne 0 -o -z "${DATE}" ]; then
		continue
	fi
	lines=$(cat $dir/$file | wc -l)
	echo "$friend $lines"
done 
} | awk '{ cnt[$1]+=$2 } END { for( i in cnt) printf("%s %s\n",i,cnt[i]) }' | sort -n -r -k 2 | head -n 10
```

--------------------------------------------------------
26 pdf 


```
#!/bin/bash

if [ $# -ne 2 ]; then
	echo "Invalid number of arguments!"
	exit 1
fi

if [ ! -r $1 ]; then
	echo "First argument is invalid!"
	exit 2
fi

if [ ! -d $2 ]; then
    echo "Second argument is invalid!"
    exit 3
fi

if [ -z $(find "${2}" -maxdepth 0 -type d -empty) ];then
	"Directory is empty!"
	 exit 4
fi

num=0
while read line
do
	echo "$line;$num"
	((num++))
done < <(cat "$1" | egrep -o "^[a-zA-Z-]+ [a-zA-Z-]+" | sort -d | uniq) > "${2}/dict.txt"

while read line
do
	newnum=$(echo ${line} | cut -d ';' -f 2)
	name=$(echo ${line} | cut -d ';' -f 1)
    cat $1 | egrep "^$name[^[:alpha:]-].*" > "${2}/${newnum}.txt" 
done < <(cat "$2/dict.txt") 
```

----------------------------------------------------------------
27 pdf 

```
#!/bin/bash

if [ $# -ne 2 ] ;then
	echo "Invalid number of arguments"
	exit 1
fi

if [ ! -r "${1}" ] ;then
    echo "First argument is not readable"
    exit 2
fi

cat ${1} | sort -t "," -n -r -k 1,1 | awk -F ',' 'BEGIN {OFS=FS} {id=$1; $1=""; cnt[$0]=id; }
  END {for(i in cnt) print cnt[i] i } ' > "${2}"
  ```

-------------------------------------------------------------
28 pdf

```
#!/bin/bash

CMD=$(egrep "^[+-]?[0-9]+$" | sed -E "s/^([+-]?)0*(.)/\1\2/g" | tr -d "+" | sort -n | uniq)
MINVAL=$(echo "${CMD}" | head -n 1)
MAXVAL=$(echo "${CMD}" | tail -n 1)

if [ $((-MINVAL)) -eq 0 -a $MAXVAL -eq 0 ] ;then
    echo "$MAXVAL"
elif [ $((-MINVAL)) -eq $MAXVAL -a $MAXVAL -ne 0 ]; then
    echo "$MINVAL"
    echo "$MAXVAL"
elif [ $((-MINVAL)) -gt $MAXVAL ]; then
    echo "$MINVAL"
else
    echo "$MAXVAL"
fi 

#!/bin/bash

CMD=$(egrep "^[+-]?[0-9]+$" | sed -E "s/^([+-]?)0*(.)/\1\2/g" | tr -d "+" | sort -n | uniq)

while read line
do
	NUM=$( echo "$line" | egrep -o "[0-9]+")
	if [ $NUM -eq 0 ]; then
		echo "$line $NUM"
		continue
	fi
	SUM=0
	while [ $NUM -ne 0 ]; do
	SUM=$((NUM%10+$SUM))
	NUM=$((NUM/10))
	done
	echo "$line $SUM"
done < <(echo "${CMD}") | sort -k 2,2n -k 1,1nr | tail -n 1 | awk '{ print $1 }'
```
-------------------------------------------------------------------------------------------
```
#!/bin/bash

CMD=$(egrep "^[+-]?[0-9]+$" | sed -E "s/^([+-]?)0*(.)/\1\2/g" | tr -d "+" | sort -n | uniq)

while read line
do
    NUM=$( echo "$line" | egrep -o "[0-9]+")
    SUM=$(echo $NUM | sed -E 's/([0-9])/\1+0/g'|bc)
    echo "$line $SUM"
done < <(echo "${CMD}") | sort -k 2,2n -k 1,1nr | tail -n 1 | awk '{ print $1 }'
```

---------------------------------------------------------------

29 pdf

```
#!/bin/bash

N=10

if [ "${1}" = "-n" ]; then
	if  ! echo "${2}" | egrep "^[0-9]+$" &>/dev/null; then
		echo "Invalid number!"
		exit 1
	fi
		N=${2}
		shift 2
		
fi

for i in "${@}"
do
	IDF=$(echo ${i} | sed -E "s/(\.log)$//g")
	cat ${i} | tail -n $N | while read line
	do
		TIMESTAMP=$(echo "${line}" |  cut -d ' ' -f 1,2)
		DATA=$(echo "${line}" | cut -d ' ' -f 3-)
 		echo "${TIMESTAMP} ${IDF} ${DATA}"
	done 
done | sort -t ' ' -k 1,2d
```

-----------------------------------------------------------

30 pdf 

```
#!/bin/bash

if [ $# -ne 1 ];then
	echo "Invalid number of arguments!"
	exit 1
fi

if [ ! -d "${1}" ];then
	echo "The argument is not a directory!"
	exit 2
fi 

SCRIPTTIME=$(cat temp.txt 2>/dev/null)
if [ -z $SCRIPTTIME ];then
	SCRIPTTIME=0
fi
find "${1}" -maxdepth 1 -type f -printf "%f %T@\n" 2>/dev/null | egrep -o "^[^_]+_report-[0-9]+\.tgz [0-9]+" | while read line 
do
	TIME=$( echo ${line} | cut -d ' ' -f 2)
	NAME=$( echo ${line} | cut -d ' ' -f 1)
	ONAME=$( echo ${line} | cut -d ' ' -f 1 | cut -d '_' -f 1)
	TIMESTAMP=$( echo ${line} | sed -E "s/^[^_]+_report-([0-9]+)\.tgz [0-9]+/\1/g")
	if [ $TIME -gt $SCRIPTTIME ];then
		tar -C ./extracted -x meow.txt -zf "${1}/${NAME}" --transform="s/.*/${ONAME}_$TIMESTAMP.txt/" 2>/dev/null 	
	fi
done

date +%s > temp.txt
```
