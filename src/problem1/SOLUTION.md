Provide your CLI command here:
grep '"symbol": "TSLA"' transaction-log.txt | grep '"side": "sell"' | awk -F'"' '{for(i=1;i<=NF;i++) if($i=="order_id") print $(i+2)}' | xargs -I{} sh -c 'printf "Order ID: {}\n" >> output.txt; curl -s "https://example.com/api/{}" >> output.txt; echo -e "\n--------------------------" >> output.txt'
