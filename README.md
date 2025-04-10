# IDOR-Auto

IDOR-Auto is a command-line tool written in Bash that helps automate the process of finding basic Insecure Direct Object Reference (IDOR) vulnerabilities in web applications by checking HTTP status codes for URLs constructed from a base URL and a wordlist. The tool is designed for bug bounty hunters and penetration testers as a starting point for identifying potential IDOR vulnerabilities.

## Features

* Automated potential IDOR checking using a custom wordlist.
* Supports multiple HTTP methods (GET, POST, PUT, DELETE).
* **Interactive Mode:** Pauses after each request attempt for review (default behavior).
* **Non-Interactive Mode:** Runs continuously through the wordlist.
* Option to save successful (HTTP 200) results to a file.
* Lightweight and easy to use Bash script.

## Prerequisites

Before you begin, ensure you have the following installed:

* **Bash:** A Unix shell and command language (usually pre-installed on Linux and macOS).
* **`curl`:** A command-line tool for transferring data with URLs.
* **`git`:** A distributed version control system (for cloning the repository).

## Installation

1.  Clone the repository from GitHub:
    ```bash
    git clone https://github.com/CypherNova1337/IDOR-Auto.git
    ```
2.  Navigate into the cloned directory:
    ```bash
    cd IDOR-Auto
    ```
3.  Make the script executable:
    ```bash
    chmod +x IDOR-Auto.sh
    ```

## Usage

Run the script from your terminal using command-line flags to specify the target and options.

### Syntax

```bash
./IDOR-Auto.sh -u <target_url> -w <wordlist> -m <http_method> [options]
```

Arguments

  -u <target_url>: (Required) The base URL for testing. The script will append each line from the wordlist to this URL.
        Example: If -u https://example.com/api/v1/users/ is provided, and the wordlist contains 123, the script tests https://example.com/api/v1/users/123.
        
  -w <wordlist>: (Required) Path to the file containing potential object identifiers (e.g., numbers, UUIDs, usernames), one per line.
        Example ids.txt:

        1
        2
        admin
        101
        guest

  
  -m <http_method>: (Required) The HTTP method to use (e.g., GET, POST, PUT, DELETE).
  
  -o <output_file>: (Optional) File path to save results. Only successful findings (HTTP 200 status code) will be appended to this file.
  
  -i <true|false>: (Optional) Controls the execution mode.
      -i true (Default if flag is omitted): Interactive Mode. The script pauses after each URL check, prompts for Enter key press to continue, and displays the status code for all attempts ([+] for 200, [-] for others).
      -i false: Non-Interactive Mode. The script runs continuously through the entire wordlist without pausing. It only prints successful (HTTP 200) results ([+]) to the console.
    
  -h: Display this help menu and exit.

Examples

  Basic Interactive Scan using GET:

./IDOR-Auto.sh -u https://api.example.com/items/ -w item_ids.txt -m GET

(This runs interactively, pausing after each check)

Non-Interactive POST Scan, Saving Successes:


./IDOR-Auto.sh -u https://api.example.com/profiles/ -w usernames.txt -m POST -o successful_posts.txt -i false

(This runs without pausing and saves only HTTP 200 results to successful_posts.txt)

Interactive PUT Scan:

  ./IDOR-Auto.sh -u https://api.internal/resource/ -w resource_ids.txt -m PUT -i true

## How it Works & Disclaimer



 **Here's a more detailed breakdown of what the script does:**

  Argument Parsing:
      The script uses the built-in Bash command getopts to parse the command-line flags (-u, -w, -m, -o, -i, -h) you provide.
      It assigns the values you pass (like the URL, wordlist path, etc.) to internal variables.
      It checks if the required arguments (-u, -w, -m) were provided. If not, it prints an error, shows the help menu, and exits.

  Reading the Wordlist:
      The script reads the file specified by the -w flag line by line using a while IFS= read -r line loop. This method ensures each line is read exactly as it appears, including potential leading/trailing whitespace if not handled otherwise (though typically IDs don't have this issue).

  URL Construction:
      Inside the loop, for each line read from the wordlist, it constructs the full URL to test by simply appending the line to the base URL provided via the -u flag (full_url="${url}/${line}").

  Making the HTTP Request:
      The core of the testing is done using the curl command:
      
  response=$(curl -s -o /dev/null -w "%{http_code}" -X ${method} ${full_url})

   -s: Runs curl in silent mode (no progress bars).
   
   -o /dev/null: Discards the actual response body (the HTML, JSON, etc.). We only need the status code for this basic check.
   
   -w "%{http_code}": This is the key part – it tells curl to output only the HTTP status code (like 200, 404, 403, etc.) after the request completes. This output is captured into the response variable.
   
   -X ${method}: Sets the HTTP method (GET, POST, etc.) based on the value you provided with the -m flag.
   
   ${full_url}: The complete URL being tested in the current loop iteration.

Analyzing the Response:

  The script checks the value stored in the response variable (which contains the HTTP status code):
  
  if [[ ${response} -eq 200 ]]; then ... else ... fi

  If the status code is exactly 200, it's considered a potential finding.

  Output and File Saving:
        Success (HTTP 200):
            It prints a line to the console starting with [+] Potential IDOR vulnerability found:, followed by the URL and status code.
            If you specified an output file using -o <output_file>, it appends the URL and status code (${full_url} (HTTP ${response})) to that file.
        Other Status Codes (Non-200):
            These results ([-] ${full_url} (HTTP ${response})) are only printed to the console if the script is running in Interactive Mode (-i true, which is the default). Non-interactive mode skips printing these.

  Interactive vs. Non-Interactive Execution:
        The main difference lies in the scan_idor_interactive function versus scan_idor_non_interactive.
        Interactive: Contains read -p "Press Enter to continue..." inside the loop, causing the script to pause after every single request until you press Enter. It also prints results for all status codes.
        Non-Interactive: Omits the read command and the else block for printing non-200 results, allowing it to cycle through the entire wordlist without stopping and only showing successes.

Disclaimer

Important: An HTTP 200 response simply indicates the URL was accessible and returned a success code. It does NOT automatically confirm a security vulnerability. Many endpoints will correctly return 200 for valid object IDs that you are supposed to access.  

Results from this tool must be manually verified (e.g., by visiting the URL in a browser, examining the response in a tool like Burp Suite) to determine if access to the object should have been permitted. Check if you are accessing resources belonging to other users or sensitive data that your account should not have access to. This tool is a basic endpoint discovery and status checker; treat its findings as leads for further investigation, not confirmed vulnerabilities. It may produce many false positives depending on the target application.
Contributing

If you find a bug, have suggestions for improvement, or want to contribute to the project, please feel free to open an issue or submit a pull request on the project's GitHub page: https://github.com/CypherNova1337/IDOR-Auto  

License

IDOR-Auto is licensed under the MIT license. See the LICENSE file for more information.  


This expanded section provides much more detail on the script's internal workings, which should make it clearer how the tool operates from start to finish.

