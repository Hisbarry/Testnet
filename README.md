# Testnet
#!/bin/bash

# Set the log file and password file
LOG_FILE=/var/log/user_management.log
PASSWORD_FILE=/var/secure/user_passwords.txt

# Check if the input file is provided
if [ $# -ne 1 ]; then
  echo "Usage: $0 <input_file>"
  exit 1
fi

# Read the input file
INPUT_FILE=$1

# Create the log file and password file if they don't exist
touch $LOG_FILE
chmod 600 $LOG_FILE
touch $PASSWORD_FILE
chmod 600 $PASSWORD_FILE

# Iterate over each line in the input file
while IFS=';' read -r user groups; do
  # Remove whitespace from the user and groups
  user=$(echo "$user" | tr -d '[:space:]')
  groups=$(echo "$groups" | tr -d '[:space:]')

  # Create the user's personal group
  groupadd $user

  # Create the user and add them to their personal group
  useradd -m -g $user -s /bin/bash $user

  # Add the user to the specified groups
  IFS=',' read -r -a group_array <<< "$groups"
  for group in "${group_array[@]}"; do
    groupadd $group
    usermod -aG $group $user
  done

  # Generate a random password for the user
  password=$(pwgen -s 12 1)

  # Set the user's password
  echo "$user:$password" | chpasswd

  # Log the action
  echo "Created user $user with password $password and added to groups ${group_array[*]}" >> $LOG_FILE

  # Store the password securely
  echo "$user,$password" >> $PASSWORD_FILE
done < $INPUT_FILE
