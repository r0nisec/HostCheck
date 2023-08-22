#!/bin/bash

# Function to display help information
function display_help() {
    echo ""
    echo " ██░ ██  ▒█████    ██████ ▄▄▄█████▓    ▄████▄   ██░ ██ ▓█████  ▄████▄   ██ ▄█▀"
    echo "▓██░ ██▒▒██▒  ██▒▒██    ▒ ▓  ██▒ ▓▒   ▒██▀ ▀█  ▓██░ ██▒▓█   ▀ ▒██▀ ▀█   ██▄█▒ "
    echo "▒██▀▀██░▒██░  ██▒░ ▓██▄   ▒ ▓██░ ▒░   ▒▓█    ▄ ▒██▀▀██░▒███   ▒▓█    ▄ ▓███▄░ "
    echo "░▓█ ░██ ▒██   ██░  ▒   ██▒░ ▓██▓ ░    ▒▓▓▄ ▄██▒░▓█ ░██ ▒▓█  ▄ ▒▓▓▄ ▄██▒▓██ █▄ "
    echo "░▓█▒░██▓░ ████▓▒░▒██████▒▒  ▒██▒ ░    ▒ ▓███▀ ░░▓█▒░██▓░▒████▒▒ ▓███▀ ░▒██▒ █▄"
    echo " ▒ ░░▒░▒░ ▒░▒░▒░ ▒ ▒▓▒ ▒ ░  ▒ ░░      ░ ░▒ ▒  ░ ▒ ░░▒░▒░░ ▒░ ░░ ░▒ ▒  ░▒ ▒▒ ▓▒"
    echo " ▒ ░▒░ ░  ░ ▒ ▒░ ░ ░▒  ░ ░    ░         ░  ▒    ▒ ░▒░ ░ ░ ░  ░  ░  ▒   ░ ░▒ ▒░"
    echo " ░  ░░ ░░ ░ ░ ▒  ░  ░  ░    ░         ░         ░  ░░ ░   ░   ░        ░ ░░ ░ "
    echo " ░  ░  ░    ░ ░        ░              ░ ░       ░  ░  ░   ░  ░░ ░      ░  ░   "
    echo "                                      ░                       ░               "
    echo ""
    echo "Usage: source $0 [-t <threads>] [-c] [-e] [-ip]"
    echo "Options:"
    echo "  -h             Display this help message"
    echo "  -t <threads>   Number of threads for parallel resolution (default is 4)"
    echo "  -c             Search for the alias domains"
    echo "  -e             Filter SERVFAIL or REFUSED hosts"
    echo "  -ip            Point your domain and subdomains to an IP address."
}

# Function to validate a domain name
function validate_domain() {
    domain=$1
    # Validate the domain using a regular expression
    if ! [[ "$domain" =~ ^[a-zA-Z0-9.-]+$ ]]; then
        echo "Invalid domain name: $domain"
        exit 1
    fi
}

# Function to resolve a domain using the host command
function resolve_domain() {
    domain=$1
    validate_domain "$domain"

    if [ "$grep_alias" = true ]; then
        # Execute the host command and use grep to search for the word "alias" only
        host "$domain" | grep -w "alias" | cut -d ' ' -f 6 | sed 's/\.$//'
    else
        # Execute the host command and grep for the "IP" word only if -ip option is provided
        if [ "$grep_ip" = true ]; then
            host "$domain" | grep -w "has\ address" | cut -d ' ' -f 4
        else
            host "$domain"
        fi
    fi
}
# Function to resolve a domain and filter lines with SERVFAIL or REFUSED words
function resolve_domain_with_filter() {
    domain=$1
    resolve_output=$(resolve_domain "$domain")

    # Check if the resolve_output contains SERVFAIL or REFUSED
    if echo "$resolve_output" | grep -E "(SERVFAIL|REFUSED)"; then
        echo "$resolve_output"
    fi
}

# Check if the script is being sourced or executed
if [ "$(basename "$0")" == "script_name.sh" ]; then
    # Check if a domain name is provided as an argument
    if [ $# -eq 0 ]; then
        echo "Usage: $0 <domain>"
        exit 1
    fi

    domain=$1
    resolve_domain "$domain"
    exit
fi

# If the script is being sourced, no domain argument is needed.
# Instead, the script will read domains from stdin and resolve them in parallel.

# Number of threads (default is 4)
threads=4

# Variable to store if we should use grep for 'alias' or not
grep_alias=false

# Variable to store if we should filter lines with SERVFAIL or REFUSED words
filter_errors=false

# Parse command-line options
while getopts "hct:eip" opt; do
    case $opt in
        h)
            display_help
            exit 0
            ;;
        c)
            grep_alias=true
            ;;
        t)
            threads=$OPTARG
            ;;
        e)
            filter_errors=true
            ;;
        i)
            grep_ip=true
            ;;
        \?)
            display_help
            exit 1
            ;;
    esac
done
# Function to read domains from stdin and resolve them in parallel
function resolve_domains_in_parallel() {
    while read -r domain; do
        if [ "$filter_errors" = true ]; then
            (resolve_domain_with_filter "$domain") &
        else
            (resolve_domain "$domain") &
        fi
    done
    wait
}

# Use function to resolve domains in parallel
resolve_domains_in_parallel
