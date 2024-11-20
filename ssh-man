#!/bin/bash

HOST="$1"
INVENTORY="PATH-TO-ANSIBLE-INVENTORY"

preflight_check() {

	if [ -z "$HOST" ]; then
		echo "Usage: $0 hostname"
		exit
	fi

	if [ ! -f "$INVENTORY" ]; then
		echo "Inventory file not found in $INVENTORY"
		exit
	fi
}

prerun_check() {

	STRING=`grep ^$HOST $INVENTORY`

	if [ -z "$STRING" ]; then
		echo "$HOST not found in $INVENTORY"
		exit
	fi

	STRING_COUNT=$(echo "$STRING" | wc -l)

	if [ "$STRING_COUNT" -ne 1 ]; then
	    echo "Many host's found by this name: $HOST"
	    exit
	fi
}

populate_vars() {

	SSHIP=$(echo "$STRING" | awk '{for(i=1;i<=NF;i++) if($i~/ansible_ssh_host=/) print $i}' | cut -d'=' -f2)
	SSHPORT=$(echo "$STRING" | awk '{for(i=1;i<=NF;i++) if($i~/ansible_ssh_port=/) print $i}' | cut -d'=' -f2)
	SSHUSER=$(echo "$STRING" | awk '{for(i=1;i<=NF;i++) if($i~/ansible_ssh_user=/) print $i}' | cut -d'=' -f2)
	SSHPASS=$(echo "$STRING" | awk '{for(i=1;i<=NF;i++) if($i~/ansible_ssh_pass=/) print $i}' | cut -d'=' -f2)
	SSHBECOME=$(echo "$STRING" | awk '{for(i=1;i<=NF;i++) if($i~/ansible_become_method=/) print $i}' | cut -d'=' -f2)

	if [ -z "$SSHIP" ]; then
		echo "No IP in $INVENTORY"
		exit
	fi

	if [ -z "$SSHPORT" ]; then
		SSHPORT="22"
	fi

	if [ -z "$SSHUSER" ]; then
		echo "No user in $INVENTORY"
		exit
	fi

	if [ -z "$SSHPASS" ]; then
		echo "No password in $INVENTORY"
		exit
	fi

	if [ -z "$SSHBECOME" ]; then
		SSHBECOME="no"
	fi
}

become_no() {

	expect -c "
	set timeout 10
	spawn ssh -l $SSHUSER -p $SSHPORT $SSHIP
	expect {
    		\"yes/no\" { send \"yes\r\"; exp_continue }
    		\"password:\" { send \"$SSHPASS\r\" }
	}

	interact
	"
}

become_sudo() {

	expect -c "
	set timeout 10
	spawn ssh -l $SSHUSER -p $SSHPORT $SSHIP
	expect {
		 \"yes/no\" { send \"yes\r\"; exp_continue }
		\"password:\" { send \"$SSHPASS\r\" }
	}

	expect \"\$ \"
	send \"sudo -s\r\"
	expect \"password for $SSHUSER:\"
	send \"$SSHPASS\r\"
	interact
	"
}

login() {

	case $SSHBECOME in
		"no")
			become_no;;
		"sudo")
			become_sudo;;
	esac
}

preflight_check
prerun_check
populate_vars
login



# EOF
