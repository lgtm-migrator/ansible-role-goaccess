[![Version on Galaxy](https://img.shields.io/badge/available%20on%20ansible%20galaxy-jonaspammer.goaccess-brightgreen)](https://galaxy.ansible.com/jonaspammer/goaccess) [![Testing CI](https://github.com/JonasPammer/ansible-role-goaccess/actions/workflows/ci.yml/badge.svg)](https://github.com/JonasPammer/ansible-role-goaccess/actions/workflows/ci.yml)

An Ansible role that

- installs GoAccess, a real-time web log analyzer that runs in a terminal or the browser

- generates a full GoAccess configuration file (at a given location)

- (optionally) generates a systemd service to generate a real-time (continiously updated) html using goaccess

- (optional) makes sure permissions of all files involved are correct

# 🔎 Metadata

Below you can find information on…

- the role’s required Ansible version

- the role’s supported platforms

- the role’s [role dependencies](https://docs.ansible.com/ansible/latest/user_guide/playbooks_reuse_roles.html#role-dependencies)

**[meta/main.yml](meta/main.yml)**

    ---
    galaxy_info:
      role_name: "goaccess"
      description: "An ansible role for installing GoAccess, a real-time web log analyzer that runs in a terminal or the browser."
      standalone: true

      author: "jonaspammer"
      license: "MIT"

      min_ansible_version: "2.11"
      platforms:
        - name: EL # (Enterprise Linux)
          versions:
            - "8" # actively tested: rockylinux8, centos8
        - name: Fedora
          versions:
            - "35"
        - name: Debian
          versions:
            - buster # debian10 (actively tested)
            - bullseye # debian11 (actively tested)
        - name: Ubuntu
          versions:
            - xenial # ubuntu1604 (actively tested)
            - bionic # ubuntu1804 (actively tested)
            - focal # ubuntu2004 (actively tested)

      galaxy_tags: []

    dependencies: []

    allow_duplicates: true

# 📌 Requirements

The Ansible User needs to be able to `become`.

The [`community.general` collection](https://galaxy.ansible.com/community/general) must be installed on the Ansible controller.

# 📜 Role Variables

    goaccess_dependency_packages: [OS-dependant by default, see /defaults directory]

List of packages to install from the system’s package manager. See [GoAccess Official Documentation](https://github.com/allinurl/goaccess#distribution-packages) for reference.

    goaccess_install_method: "{{ 'source' if ansible_os_family != 'RedHat' else 'system' }}"

One of “source” or “system”.

I could not get install from source method to work for RedHat because of error as seen in [this CI run](https://github.com/JonasPammer/ansible-role-goaccess/runs/7031791748?check_suite_focus=true):

>       TASK [ansible-role-goaccess : Execute autoreconf.] *****************************
>       fatal: [instance-py3-ansible-5-fedora35]: FAILED! => changed=false
>         cmd:
>         - autoreconf
>         - --force
>         - --install
>         - --verbose
>         delta: '0:00:00.764053'
>         msg: non-zero return code
>         rc: 1
>         stderr: |-
>           autoreconf: Entering directory `.'
>           autoreconf: running: autopoint --force
>           /usr/bin/autopoint: line 498: find: command not found
>           autopoint: *** infrastructure files for version 0.19 not found; this is autopoint from GNU gettext-tools 0.21
>           autopoint: *** Stop.
>           autoreconf: autopoint failed with exit status: 1

As the goaccess in RockyLinux 8’s repository is actually the latest avaiable as of writing this (2022/07) I do not see it as a problem at all.

Pull Requests or Issues with Solutions from Wizards of the C world are welcome as always if.

    goaccess_command_dir: "{{ '/usr/local/bin' if goaccess_install_method == 'source' else '/usr/bin' }}"

Directory of `goaccess` binary. Used in systemd and for source installation method version check.

## Role Variables used by the `system`-installation-method

    goaccess_system_install_official_repo: true

(Debian/Ubuntu only) Wheter to install [ GoAccess’s Official APT Repository](https://goaccess.io/download#official-repo). This is used to get a more recent version than the one packaged in the system itself.

    goaccess_system_package_state: present

When using `goaccess_system_install_official_repo` you can change this to “latest” to ensure that this role installs the latest available `goaccess` from the system repository.

## Role Variables used by the `source`-installation-method

    goaccess_source_version: "v{{ goaccess_version }}"

The [git version](https://github.com/allinurl/goaccess/tags) to download.

    goaccess_version: 1.6

The goaccess version string to check against.

    goaccess_source_dependency_packages: [OS-dependant by default, see /defaults directory]

List of packages to install from the system’s package manager.

    goaccess_source_configure_parameters: "--enable-utf8 --enable-geopip=mmdb"

Build Configuration Arguments to pass to `./configure` (autoconf) which creates the `Makefile` and `src/config.h` (and others).

The Default Options result in the following summary:

      Prefix         : /usr/local
      Package        : goaccess
      Version        : 1.6
      Compiler flags :  -pthread
      Linker flags   : -lnsl -lncursesw -lmaxminddb -lpthread
      UTF-8 support  : yes
      Dynamic buffer : no
      Geolocation    : GeoIP2
      Storage method : In-Memory with On-Disk Persistent Storage
      TLS/SSL        : no
      Bugs           : hello@goaccess.io

These options are also displayed when executing `goaccess --version`.

## Role Variables for creating a GoAccess Configuration File

    goaccess_conf_file: "/etc/goaccess.conf"
    goaccess_conf_file_owner: root
    goaccess_conf_file_group: root
    goaccess_conf_file_mode: u=rw,g=r,o=

Location of the `.goaccess` file to generate. [Can be](https://goaccess.io/man#options) in the home directory of a user.

Below you can find Configuration Options used in goaccess' configuration file template. Associated Text for each variable has mostly been taken from [ the official GoAccess Git’s example "config/goaccess.conf"](https://github.com/allinurl/goaccess/blob/master/config/goaccess.conf) and added here too as a convenience.

Normal-Properties with value of `None` (`~`) as well as Array-Properties with size of 0 (`[]`) will not be inserted-into/used-in the Template.

### Time Format

    goaccess_conf_time_format: "%H:%M:%S" # (Default used by Apache/NGINX's log format Added by role-author)

> **Required.**
>
> The hour (24-hour clock) \[00,23\]; leading zeros are permitted but not required.
> The minute \[00,59\]; leading zeros are permitted but not required.
> The seconds \[00,60\]; leading zeros are permitted but not required.
> See `man strftime` for more details
>
> Other examples:
>
> **Google Cloud Storage or The time in microseconds since the Unix epoch**
>
>     goaccess_conf_time_format: %f
>
> **Squid native log format**
>
>     goaccess_conf_time_format: %s

The default time format works with any of the Apache/NGINX’s log formats denoted in the description of [programlisting_title](#goaccess_conf_log_format).

### Date Format

    goaccess_conf_date_format: "%d/%b/%Y" # (Default used by Apache/NGINX's log format Added by role-author)

> **Required.**
>
> The date-format variable followed by a space, specifies the log format date containing any combination of regular characters and special format specifiers. They all begin with a percentage (%) sign.
> See `man strftime`
>
> Other examples:
>
> **AWS Amazon CloudFront (Download Distribution), AWS Elastic Load Balancing, W3C (IIS)**
>
>     goaccess_conf_date_format: "%Y-%m-%d"
>
> **Google Cloud Storage or The time in microseconds since the Unix epoch.**
>
>     goaccess_conf_date_format: "%f"
>
> **Squid native log format, Caddy**
>
>     goaccess_conf_date_format: "%s"

The default time format works with any of the Apache/NGINX’s log formats denoted in the description of [programlisting_title](#goaccess_conf_log_format).

### Log Format

    goaccess_conf_log_format: COMMON # (Default Added by role-author)

> The log-format variable followed by a space or \t for tab-delimited, specifies the log format string.
>
> If the time/date is a timestamp in seconds or microseconds %x must be used instead of %d & %t to represent the date & time.
>
> **NCSA Combined Log Format**
>
>     goaccess_conf_log_format: '%h %^[%d:%t %^] "%r" %s %b "%R" "%u"'
>
> **NCSA Combined Log Format with Virtual Host**
>
>     goaccess_conf_log_format: '%v:%^ %h %^[%d:%t %^] "%r" %s %b "%R" "%u"'
>
> **Common Log Format (CLF)**
>
>     goaccess_conf_log_format: '%h %^[%d:%t %^] "%r" %s %b'
>
> **Common Log Format (CLF) with Virtual Host**
>
>     goaccess_conf_log_format: '%v:%^ %h %^[%d:%t %^] "%r" %s %b'
>
> **W3C**
>
>     goaccess_conf_log_format: '%d %t %h %^ %^ %^ %^ %r %^ %s %b %^ %^ %u %R'
>
> **Squid native log format**
>
>     goaccess_conf_log_format: '%^ %^ %^ %v %^: %x.%^ %~%L %h %^/%s %b %m %U'
>
> **AWS | Amazon CloudFront (Download Distribution)**
>
>     goaccess_conf_log_format: '%d\t%t\t%^\t%b\t%h\t%m\t%^\t%r\t%s\t%R\t%u\t%^'
>
> **Google Cloud Storage**
>
>     goaccess_conf_log_format: '"%x","%h",%^,%^,"%m","%U","%s",%^,"%b","%D",%^,"%R","%u"'
>
> **AWS | Elastic Load Balancing**
>
>     goaccess_conf_log_format: '%dT%t.%^ %^ %h:%^ %^ %T %^ %^ %^ %s %^ %b "%r" "%u"'
>
> **AWSS3 | Amazon Simple Storage Service (S3)**
>
>     goaccess_conf_log_format: '%^[%d:%t %^] %h %^"%r" %s %^ %b %^ %L %^ "%R" "%u"'
>
> **Virtualmin Log Format with Virtual Host**
>
>     goaccess_conf_log_format: '%h %^ %v %^[%d:%t %^] "%r" %s %b "%R" "%u"'
>
> **Kubernetes Nginx Ingress Log Format**
>
>     goaccess_conf_log_format: '%^ %^ [%h] %^ %^ [%d:%t %^] "%r" %s %b "%R" "%u" %^ %^ [%v] %^:%^ %^ %T %^ %^'
>
> **CADDY JSON Structured**
>
>     goaccess_conf_log_format: '{ts:"%x.%^",request:{remote_ip:"%h",proto:"%H",method:"%m",host:"%v",uri:"%U",headers:{"User-Agent":["%u","%^"]},tls:{cipher_suite:"%k",proto:"%K"}},duration:"%T",size:"%b",status:"%s",resp_headers:{"Content-Type":["%M;%^"]}}'
>
> In addition to specifying the raw log/date/time formats, for simplicity, any of the following predefined log format names can be supplied to the log/date/time-format variables. GoAccess can also handle one predefined name in one variable and another predefined name in another variable.
>
>     goaccess_conf_log_format: COMBINED
>     goaccess_conf_log_format: VCOMBINED
>     goaccess_conf_log_format: COMMON
>     goaccess_conf_log_format: VCOMMON
>     goaccess_conf_log_format: W3C
>     goaccess_conf_log_format: SQUID
>     goaccess_conf_log_format: CLOUDFRONT
>     goaccess_conf_log_format: CLOUDSTORAGE
>     goaccess_conf_log_format: AWSELB
>     goaccess_conf_log_format: AWSS3
>     goaccess_conf_log_format: CADDY

### UI Options

    goaccess_conf_color_scheme: 2 # (Default Added by role-author)

> Choose among color schemes
>
> 1
> Monochrome
>
> 2
> Green
>
> 3
> Monokai (if 256-colors supported)

    goaccess_conf_config_dialog: false

> _Boolean._ Prompt log/date configuration window on program start.

    goaccess_conf_hl_header: true

> _Boolean._ Color highlight active panel.

    goaccess_conf_html_custom_css: ~

> Specify a custom CSS file in the HTML report.

    goaccess_conf_html_custom_js: ~

> Specify a custom JS file in the HTML report.

    goaccess_conf_html_prefs: ~

> Set default HTML preferences.
>
> A valid JSON object is required. DO NOT USE A MULTILINE JSON OBJECT. The parser will only parse the value next to `html-prefs` (single line) It allows the ability to customize each panel plot. See example below.
>
>     goaccess_conf_html_prefs: '{"theme":"bright","perPage":5,"layout":"horizontal","showTables":true,"visitors":{"plot":{"chartType":"bar"}}}'

    goaccess_conf_html_report_title: ~

> _String._ Set HTML report page title and header.

    goaccess_conf_json_pretty_print: true # (Default Changed by role-author)

> _Boolean._ Format JSON output using tabs and newlines.

    goaccess_conf_no_color: false

> _Boolean._ Whether to turn off colored output. This is the default output on terminals that do not support colors.
>
> true
> no color output
>
> false
> use color-scheme

    goaccess_conf_no_column_names: false

> _Boolean._ Whether to write column names in the terminal output. By default, it displays column names for each available metric in every panel.

    goaccess_conf_no_csv_summary: false

> _Boolean._ Disable summary metrics on the CSV output.

    goaccess_conf_no_progress: false

> _Boolean._ Disable progress metrics.

    goaccess_conf_no_tab_scroll: false

> _Boolean._ Disable scrolling through panels on TAB.

    goaccess_conf_no_parsing_spinner: ~

> _Boolean._ Disable progress metrics and parsing spinner.

    goaccess_conf_no_html_last_updated: ~

> _Boolean._ Do not show the last updated field displayed in the HTML generated report.

    goaccess_conf_with_mouse: true # (Default Changed by role-author)

> _Boolean._ Enable mouse support on main dashboard.

    goaccess_conf_max_items: ~

> Maximum number of items to show per panel.
>
> Only the CSV and JSON outputs allow a maximum greater than the default value of 366.

    goaccess_conf_colors: []

> _Array of strings._
>
> Custom colors for the terminal output.
>
> Color syntax:
> `DEFINITION space/tab colorFG#:colorBG# [[attributes,] PANEL]`
>
> - `FG#` = foreground color number \[-1…​255\] (-1 = default terminal color)
>
> - `BG#` = background color number \[-1…​255\] (-1 = default terminal color)
>
> Optionally:
>
> It is possible to apply color attributes, such as: bold,underline,normal,reverse,blink. Multiple attributes are comma separated
>
> If desired, it is possible to apply custom colors per panel, that is, a metric in the REQUESTS panel can be of color A, while the same metric in the BROWSERS panel can be of color B.
>
> The following is a 256 color scheme (hybrid palette):
>
>     goaccess_conf_colors:
>       - "MTRC_HITS              color110:color-1"
>       - "MTRC_VISITORS          color173:color-1"
>       - "MTRC_DATA              color221:color-1"
>       - "MTRC_BW                color167:color-1"
>       - "MTRC_AVGTS             color143:color-1"
>       - "MTRC_CUMTS             color247:color-1"
>       - "MTRC_MAXTS             color186:color-1"
>       - "MTRC_PROT              color109:color-1"
>       - "MTRC_MTHD              color139:color-1"
>       - "MTRC_HITS_PERC         color186:color-1"
>       - "MTRC_HITS_PERC_MAX     color139:color-1"
>       - "MTRC_HITS_PERC_MAX     color139:color-1 VISITORS"
>       - "MTRC_HITS_PERC_MAX     color139:color-1 OS"
>       - "MTRC_HITS_PERC_MAX     color139:color-1 BROWSERS"
>       - "MTRC_HITS_PERC_MAX     color139:color-1 VISIT_TIMES"
>       - "MTRC_VISITORS_PERC     color186:color-1"
>       - "MTRC_VISITORS_PERC_MAX color139:color-1"
>       - "PANEL_COLS             color243:color-1"
>       - "BARS                   color250:color-1"
>       - "ERROR                  color231:color167"
>       - "SELECTED               color7:color167"
>       - "PANEL_ACTIVE           color7:color237"
>       - "PANEL_HEADER           color250:color235"
>       - "PANEL_DESC             color242:color-1"
>       - "OVERALL_LBLS           color243:color-1"
>       - "OVERALL_VALS           color167:color-1"
>       - "OVERALL_PATH           color186:color-1"
>       - "ACTIVE_LABEL           color139:color235 bold underline"
>       - "BG                     color250:color-1"
>       - "DEFAULT                color243:color-1"
>       - "PROGRESS               color7:color110"

### Server Options

    goaccess_conf_addr: ~

> Specify IP address to bind server to.
>
> **Example**
>
>     goaccess_conf_addr: 0.0.0.0

    goaccess_conf_daemonize: ~

> _Boolean._ Run GoAccess as daemon (if --real-time-html enabled).

    goaccess_conf_origin: ~

> Ensure clients send the specified origin header upon the WebSocket handshake.
>
> **Example**
>
>     goaccess_origin: http://example.org

    goaccess_conf_port: ~

> The port to which the connection is being attempted to connect. By default GoAccess' WebSocket server listens on port 7890 See man page or <http://gwsocket.io> for details.
>
> **Example**
>
>     goaccess_conf_port: 7890

    goaccess_conf_pid_file: ~

> Write the PID to a file when used along the daemonize option.
>
> **Example**
>
>     goaccess_conf_pid_file: /var/run/goaccess.pid

    goaccess_conf_real_time_html: "{{ goaccess_systemd }}"

> _Boolean._ Enable real-time HTML output.

    goaccess_conf_ssl_cert: ~

> Path to TLS/SSL certificate.
>
> Note that ssl-cert and ssl-key need to be used to enable TLS/SSL.

    goaccess_conf_ssl_key: ~

> Path to TLS/SSL private key.
>
> Note that ssl-cert and ssl-key need to be used to enable TLS/SSL.

    goaccess_conf_ws_url: ~

> URL to which the WebSocket server responds. This is the URL supplied to the WebSocket constructor on the client side.
>
> Optionally, it is possible to specify the WebSocket URI scheme, such as `ws://` or `wss://` for unencrypted and encrypted connections. e.g., `goaccess_conf_ws_url: wss://goaccess.io`
>
> If GoAccess is running behind a proxy, you could set the client side to connect to a different port by specifying the host followed by a colon and the port. e.g., `goaccess_conf_ws_url: goaccess.io:9999`
>
> **By default**, it will attempt to connect to `localhost`. If GoAccess is running on a remote server, the host of the remote server should be specified here. Also, make sure it is a valid host and **NOT** an http address.
>
> **Example**
>
>     goaccess_conf_ws_url: goaccess.io

    goaccess_conf_fifo_in: ~

> Path to read named pipe (FIFO).

    goaccess_conf_fifo_out: ~

> Path to write named pipe (FIFO).

### File Options

    goaccess_conf_log_file: [OS-specific by default, see /defaults directory]

> Specify the path to the input log file. If set, it will take priority over `-f` from the command line.

    goaccess_conf_log_file_state: file
    goaccess_conf_log_file_owner: ~
    goaccess_conf_log_file_group: ~
    goaccess_conf_log_file_mode: u=rw,g=r,o=
    goaccess_conf_log_dir_alter: true
    goaccess_conf_log_dir_owner: ~
    goaccess_conf_log_dir_group: ~
    goaccess_conf_log_dir_mode: ~

This role will make sure that `goaccess_conf_log_file` as well as its directory (if `goaccess_conf_log_dir_alter` is enabled) has these configured properties set. If you do not want this role to be in charge of this you can set each of these values to None.

Note that when `goaccess_conf_log_dir_alter` is true, this role will implicitly create the directory and all intermediate subdirectory as per ansible’s file module.

    goaccess_conf_debug_file: ~

> Send all debug messages to the specified file.

    goaccess_conf_config_file: ~

> Specify a custom configuration file to use. If set, it will take priority over the global configuration file (if any).

    goaccess_conf_invalid_requests: ~

> Log invalid requests to the specified file.

    goaccess_conf_no_global_config: ~

> _Boolean._ Disable loading the global configuration file.

### Parser Options

    goaccess_conf_agent_list: true # (Default Changed by role-author)

> Enable a list of user-agents by host. For faster parsing, do not enable this flag.

    goaccess_conf_with_output_resolver: true # (Default Changed by role-author)

> Enable IP resolver on HTML|JSON|CSV output.

    goaccess_conf_exclude_ips: []

> _Array of Strings._ Exclude an IPv4 or IPv6 from being counted. Ranges can be included as well using a dash in between the IPs (start-end).
>
> **Example**
>
>     goaccess_conf_exclude_ips:
>       - "exclude-ip 127.0.0.1"
>       - "exclude-ip 192.168.0.1-192.168.0.100"
>       - "exclude-ip ::1"
>       - "exclude-ip 0:0:0:0:0:ffff:808:804-0:0:0:0:0:ffff:808:808"

    goaccess_conf_http_method: true

> _Boolean._ Include HTTP request method if found. This will create a request key containing the request method + the actual request.

    goaccess_conf_http_protocol: true

> _Boolean._ Include HTTP request protocol if found. This will create a request key containing the request protocol + the actual request.

    goaccess_conf_output: ~

> Write output to stdout given one of the following files and the corresponding extension for the output format:
>
> /path/file.csv
> Comma-separated values (CSV)
>
> /path/file.json
> JSON (JavaScript Object Notation)
>
> /path/file.html
> HTML

    goaccess_conf_no_query_string: false

> _Boolean._ Ignore request’s query string. i.e., `www.google.com/page.htm?query` ⇒ `www.google.com/page.htm`
>
> Removing the query string can greatly decrease memory consumption, especially on timestamped requests.

    goaccess_conf_no_term_resolver: false

> _Boolean._ Disable IP resolver on terminal output.

    goaccess_conf_444_as_404: false

> _Boolean._ Treat non-standard status code 444 as 404.

    goaccess_conf_4xx_to_unique_count: false

> _Boolean._ Add 4xx client errors to the unique visitors count.

    goaccess_conf_anonymize_ip: ~

> _Boolean._ Enable IP address anonymization.
>
> The IP anonymization option sets the last octet of IPv4 user IP addresses and the last 80 bits of IPv6 addresses to zeros. e.g., `192.168.20.100` ⇒ `192.168.20.0` e.g., `2a03:2880:2110:df07:face:b00c::1` ⇒ `2a03:2880:2110:df07::`

    goaccess_conf_all_static_files: false

> _Boolean._ Include static files that contain a query string in the static files panel. e.g., `/fonts/fontawesome-webfont.woff?v=4.0.3`

    goaccess_conf_browsers_file: ~

> Include an additional delimited list of browsers/crawlers/feeds etc. See [config/browsers.list](https://github.com/allinurl/goaccess/blob/master/config/browsers.list) for an example.

    goaccess_conf_date_spec: ~

> Date specificity. Possible values: `date` (default), or `hr` or `min`.

    goaccess_conf_double_decode: false

> _Boolean._ Decode double-encoded values.

    goaccess_conf_enable_panels: []

> _Array of Strings._ Enable parsing/displaying the given panels.
>
> **Example: Enable every panel**
>
>     goaccess_conf_enable_panels:
>       - VISITORS
>       - REQUESTS
>       - REQUESTS_STATIC
>       - NOT_FOUND
>       - HOSTS
>       - OS
>       - BROWSERS
>       - VISIT_TIMES
>       - VIRTUAL_HOSTS
>       - REFERRERS
>       - REFERRING_SITES
>       - KEYPHRASES
>       - STATUS_CODES
>       - REMOTE_USER
>       - CACHE_STATUS
>       - GEO_LOCATION
>       - MIME_TYPE
>       - TLS_TYPE

    goaccess_conf_hide_referers: []

> _Array of Strings._ Hide a referrer but still count it. Wild cards are allowed. i.e., `*.bing.com`
>
> **Example**
>
>     goaccess_conf_hide_referers:
>       - "*.google.com"
>       - "bing.com"

    goaccess_conf_hour_spec: ~

> Hour specificity. Possible values: `hr` (default), or `min` (tenth of a minute).

    goaccess_conf_ignore_crawlers: false

> _Boolean._
>
> Ignore crawlers from being counted. This will ignore robots listed under [`src/browsers.c`](https://github.com/allinurl/goaccess/blob/master/src/browsers.c). Note that it will count them towards the total number of requests, but excluded from any of the panels.

    goaccess_conf_crawlers_only: false

> _Boolean._ Parse and display crawlers only. This will ignore all hosts except robots listed under [`src/browsers.c`](https://github.com/allinurl/goaccess/blob/master/src/browsers.c). Note that it will count them towards the total number of requests, but excluded from any of the panels.

    goaccess_conf_ignore_statics: ~

> Ignore static file requests. Possible values:
>
> req
> Only ignore request from valid requests
>
> panels
> Ignore request from panels.
>
> Note that it will count them towards the total number of requests

    goaccess_conf_ignore_panels:
      - REFERRERS
      - KEYPHRASES

> _Array of Strings._ Ignore parsing and displaying the given panel. Opposite of [programlisting_title](#goaccess_conf_enable_panels).

    goaccess_conf_ignore_referers: []

> _Array of Strings._ Ignore referrers from being counted.
>
> This supports wild cards. For instance, '\*' matches 0 or more characters (including spaces) '?' matches exactly one character
>
> **Example**
>
>     goaccess_conf_ignore_referers:
>       - "ignore-referrer *.domain.com"
>       - "ignore-referrer ww?.domain.*"

    goaccess_conf_ignore_statuses: []

> _Array of Numbers._ Ignore parsing and displaying one or multiple status code(s)
>
> **Example**
>
>     goaccess_conf_ignore_statuses:
>       - 400
>       - 502

    goaccess_conf_keep_last: ~

> _Number._ Keep the last specified number of days in storage. This will recycle the storage tables. e.g., keep & show only the last 7 days:
>
> **Example**
>
>     goaccess_conf_keep_last: 7

    goaccess_conf_no_ip_validation: ~

> _Boolean._ Disable client IP validation. Useful if IP addresses have been obfuscated before being logged.

    goaccess_conf_num_tests: ~

> Number of lines from the access log to test against the provided log/date/time format. By default, the parser is set to test 10 lines. If set to 0, the parser won’t test any lines and will parse the whole access log.

    goaccess_conf_process_and_exit: ~

> _Boolean._ Parse log and exit without outputting data.

    goaccess_conf_real_os: true

> _Boolean._ Display real OS names. e.g, Windows XP, Snow Leopard.

    goaccess_conf_sort_panels: []

> Sort panel on initial load. Sort options are separated by comma. Options are in the form: `PANEL,METRIC,ORDER`
>
> Available metrics:
>
> BY_HITS
> Sort by hits
>
> BY_VISITORS
> Sort by unique visitors
>
> BY_DATA
> Sort by data
>
> BY_BW
> Sort by bandwidth
>
> BY_AVGTS
> Sort by average time served
>
> BY_CUMTS
> Sort by cumulative time served
>
> BY_MAXTS
> Sort by maximum time served
>
> BY_PROT
> Sort by http protocol
>
> BY_MTHD
> Sort by http method
>
> Available orders:
>
> - ASC
>
> - DESC
>
> **Example**
>
>     goaccess_conf_sort_panels:
>       - "VISITORS,BY_DATA,ASC"
>       - "REQUESTS,BY_HITS,ASC"
>       - "REQUESTS_STATIC,BY_HITS,ASC"
>       - "NOT_FOUND,BY_HITS,ASC"
>       - "HOSTS,BY_HITS,ASC"
>       - "OS,BY_HITS,ASC"
>       - "BROWSERS,BY_HITS,ASC"
>       - "VISIT_TIMES,BY_DATA,DESC"
>       - "VIRTUAL_HOSTS,BY_HITS,ASC"
>       - "REFERRERS,BY_HITS,ASC"
>       - "REFERRING_SITES,BY_HITS,ASC"
>       - "KEYPHRASES,BY_HITS,ASC"
>       - "STATUS_CODES,BY_HITS,ASC"
>       - "REMOTE_USER,BY_HITS,ASC"
>       - "CACHE_STATUS,BY_HITS,ASC"
>       - "GEO_LOCATION,BY_HITS,ASC"
>       - "MIME_TYPE,BY_HITS,ASC"
>       - "TLS_TYPE,BY_HITS,ASC"

    goaccess_conf_static_file:
      - .css
      - .js
      - .jpg
      - .png
      - .gif
      - .ico
      - .jpeg
      - .pdf
      - .csv
      - .mpeg
      - .mpg
      - .swf
      - .woff
      - .woff2
      - .xls
      - .xlsx
      - .doc
      - .docx
      - .ppt
      - .pptx
      - .txt
      - .zip
      - .ogg
      - .mp3
      - .mp4
      - .exe
      - .iso
      - .gz
      - .rar
      - .svg
      - .bmp
      - .tar
      - .tgz
      - .tiff
      - .tif
      - .ttf
      - .flv
      - .dmg
      - .xz
      - .zst # (▲ GoAccess Default)
      - .avi # (▼ Added by role-author)
      - .bz2
      - .jar
      - .ogv
      - .webm
      - .mkv
      - .ods
      - .odt
      - .wav
      - .webp

> File Extensions to consider as static files The actual '.' is required and extensions are case sensitive

### GeoIP Options

Feature Request for automating this using this role tracked in <https://github.com/JonasPammer/ansible-role-goaccess/issues/2>

> To feed a database either through GeoIP Legacy or GeoIP2, you need to use the geoip-database flag below.
>
> GeoIP Legacy
> Legacy GeoIP has been discontinued. If your GNU+Linux distribution does not ship with the legacy databases, you may still be able to find them through different sources. Make sure to download the .dat files. Distributed with Creative Commons Attribution-ShareAlike 4.0 International License. <https://mailfud.org/geoip-legacy/>
>
> IPv4 Country database:
>
> - Download the GeoIP.dat.gz
>
> - gunzip GeoIP.dat.gz
>
> IPv4 City database:
>
> - Download the GeoIPCity.dat.gz
>
> - gunzip GeoIPCity.dat.gz

    goaccess_conf_std_geopip: ~

> _Boolean._ Activate Standard GeoIP database for less memory usage (GeoIP Legacy).

    goaccess_conf_geoip_database: ~

> _GeoIP2_. For GeoIP2 databases, you can use DB-IP Lite databases. DB-IP is licensed under a Creative Commons Attribution 4.0 International License. <https://db-ip.com/db/lite.php>
>
> Or you can download them from MaxMind <https://dev.maxmind.com/geoip/geoip2/geolite2/>
>
> For GeoIP2 City database: \* Download the GeoLite2-City.mmdb.gz \* gunzip GeoLite2-City.mmdb.gz
>
> For GeoIP2 Country database: \* Download the GeoLite2-Country.mmdb.gz \* gunzip GeoLite2-Country.mmdb.gz
>
> **Example**
>
>     goaccess_conf_geoip_database: /usr/local/share/GeoIP/GeoLiteCity.dat

### Persistence Options

    goaccess_conf_db_path: ~

> Path where the persisted database files are stored on disk. The default value is the `/tmp` directory.

    goaccess_conf_persist: ~

> _Boolean._ Persist parsed data into disk.

    goaccess_conf_restore: ~

> Load previously stored data from disk. Database files need to exist. See `persist`.

### Role Variables for creating a systemd service

Fails on CentOS 7 because of too old goaccess version in system package manager (which is the default install method because of the problem described in [goaccess_install_method](#goaccess_install_method-redhat_notice)).

This service only works if you’ve correctly filled-in GoAccess’s Configuration File so it starts without error or interuption when called with `--real-time-html`.

    goaccess_systemd: false

Toggle this feature.

    goaccess_conf_file_owner: root
    goaccess_conf_file_group: root
    goaccess_conf_file_mode: u=rw,g=r,o=

Systemd Unit and File Permissions Options.

    goaccess_systemd_name: "goaccess-{{ goaccess_conf_file_owner }}"
    goaccess_systemd_description: "Service which generates real-time-html reports of {{ goaccess_conf_log_file }} using GoAccess"

Systemd Unit Options.

    goaccess_systemd_html_output_location: "/var/www/html/{{ goaccess_systemd_name }}.html"

Path passed to `goaccess --real-time-html`

# 📜 Facts/Variables defined by this role

Each variable listed in this section is dynamically defined when executing this role (and can only be overwritten using `ansible.builtin.set_facts`) _and_ is meant to be used not just internally.

# 🏷️ Tags

Tasks are tagged with the following [tags](https://docs.ansible.com/ansible/latest/user_guide/playbooks_tags.html#adding-tags-to-roles):

<table>
<colgroup>
<col style="width: 50%" />
<col style="width: 50%" />
</colgroup>
<thead>
<tr class="header">
<th style="text-align: left;">Tag</th>
<th style="text-align: left;">Purpose</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td colspan="2" style="text-align: left;"><p>This role does not have officially documented tags yet.</p></td>
</tr>
</tbody>
</table>

You can use Ansible to skip tasks, or only run certain tasks by using these tags. By default, all tasks are run when no tags are specified.

# 👫 Dependencies

# 📚 Example Playbook Usages

This role is part of [ many compatible purpose-specific roles of mine](https://github.com/JonasPammer/ansible-roles).

The machine needs to be prepared. In CI, this is done in `molecule/resources/prepare.yml` which sources its soft dependencies from `requirements.yml`:

**[molecule/resources/prepare.yml](molecule/resources/prepare.yml)**

    ---
    - name: prepare
      hosts: all
      become: true
      gather_facts: false

      vars:
        apache_vhosts:
          - servername: "localhost"
            documentroot: "/var/www/html"

      roles:
        - name: jonaspammer.bootstrap
        - name: jonaspammer.apache2
        #    - name: jonaspammer.core_dependencies

The following diagram is a compilation of the "soft dependencies" of this role as well as the recursive tree of their soft dependencies.

![requirements.yml dependency graph of jonaspammer.goaccess](https://raw.githubusercontent.com/JonasPammer/ansible-roles/master/graphs/dependencies_goaccess.svg)

    roles:
      - jonaspammer.goaccess

    vars:
      some_var: "some_value"

# 📝 Development

[![Conventional Commits](https://img.shields.io/badge/Conventional%20Commits-1.0.0-yellow.svg)](https://conventionalcommits.org) [![pre-commit.ci status](https://results.pre-commit.ci/badge/github/JonasPammer/ansible-role-goaccess/master.svg)](https://results.pre-commit.ci/latest/github/JonasPammer/ansible-role-goaccess/master)

## 📌 Development Machine Dependencies

- Python 3.8 or greater

- Docker

## 📌 Development Dependencies

Development Dependencies are defined in a [pip requirements file](https://pip.pypa.io/en/stable/user_guide/#requirements-files) named `requirements-dev.txt`. Example Installation Instructions for Linux are shown below:

    # "optional": create a python virtualenv and activate it for the current shell session
    $ python3 -m venv venv
    $ source venv/bin/activate

    $ python3 -m pip install -r requirements-dev.txt

## ℹ️ Ansible Role Development Guidelines

Please take a look at my [ Ansible Role Development Guidelines](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_GUIDELINES.adoc).

If interested, I’ve also written down some [ General Ansible Role Development (Best) Practices](https://github.com/JonasPammer/cookiecutter-ansible-role/blob/master/ROLE_DEVELOPMENT_TIPS.adoc).

## 🔢 Versioning

Versions are defined using [Tags](https://git-scm.com/book/en/v2/Git-Basics-Tagging), which in turn are [recognized and used](https://galaxy.ansible.com/docs/contributing/version.html) by Ansible Galaxy.

**Versions must not start with `v`.**

When a new tag is pushed, [ a GitHub CI workflow](https://github.com/JonasPammer/ansible-role-goaccess/actions/workflows/release-to-galaxy.yml) (![Release CI](https://github.com/JonasPammer/ansible-role-goaccess/actions/workflows/release-to-galaxy.yml/badge.svg)) takes care of importing the role to my Ansible Galaxy Account.

## 🧪 Testing

Automatic Tests are run on each Contribution using GitHub Workflows.

The Tests primarily resolve around running [Molecule](https://molecule.readthedocs.io/en/latest/) on a varying set of linux distributions and using various ansible versions, as detailed in [JonasPammer/ansible-roles](https://github.com/JonasPammer/ansible-roles).

The molecule test also includes a step which lints all ansible playbooks using [`ansible-lint`](https://github.com/ansible/ansible-lint#readme) to check for best practices and behaviour that could potentially be improved.

To run the tests, simply run `tox` on the command line. You can pass an optional environment variable to define the distribution of the Docker container that will be spun up by molecule:

    $ MOLECULE_DISTRO=centos7 tox

For a list of possible values fed to `MOLECULE_DISTRO`, take a look at the matrix defined in [.github/workflows/ci.yml](.github/workflows/ci.yml).

### 🐛 Debugging a Molecule Container

1.  Run your molecule tests with the option `MOLECULE_DESTROY=never`, e.g.:

        $ MOLECULE_DESTROY=never MOLECULE_DISTRO=ubuntu1604 tox -e py3-ansible-5
        ...
          TASK [ansible-role-pip : (redacted).] ************************
          failed: [instance-py3-ansible-5] => changed=false
        ...
         ___________________________________ summary ____________________________________
          pre-commit: commands succeeded
        ERROR:   py3-ansible-5: commands failed

2.  Find out the name of the molecule-provisioned docker container:

        $ docker ps
        30e9b8d59cdf   geerlingguy/docker-debian10-ansible:latest   "/lib/systemd/systemd"   8 minutes ago   Up 8 minutes                                                                                                    instance-py3-ansible-5

3.  Get into a bash Shell of the container, and do your debugging:

        $ docker exec -it 30e9b8d59cdf /bin/bash

        root@instance-py3-ansible-2:/#
        root@instance-py3-ansible-2:/# python3 --version
        Python 3.8.10
        root@instance-py3-ansible-2:/# ...

    If the failure you try to debug is part of `verify.yml` step and not the actual `converge.yml`, you may want to know that the output of ansible’s modules (`vars`), hosts (`hostvars`) and environment variables have been stored into files on both the provisioner and inside the docker machine under: \* `/var/tmp/vars.yml` \* `/var/tmp/hostvars.yml` \* `/var/tmp/environment.yml` `grep`, `cat` or transfer these as you wish!

    You may also want to know that the files mentioned in the admonition above are attached to the **GitHub CI Artifacts** of a given Workflow run.
    This allows one to check the difference between runs and thus help in debugging what caused the bit-rot or failure in general. image::https://user-images.githubusercontent.com/32995541/178442403-e15264ca-433a-4bc7-95db-cfadb573db3c.png\[\]

4.  After you finished your debugging, exit it and destroy the container:

        root@instance-py3-ansible-2:/# exit

        $ docker stop 30e9b8d59cdf

        $ docker container rm 30e9b8d59cdf
        or
        $ docker container prune

## 🧃 TIP: Containerized Ideal Development Environment

This Project offers a definition for a "1-Click Containerized Development Environment".

This Container even allow one to run docker containers inside of them (Docker-In-Docker, dind), allowing for molecule execution.

To use it:

1.  Ensure you fullfill the [ the System requirements of Visual Studio Code Development Containers](https://code.visualstudio.com/docs/remote/containers#_system-requirements), optionally following the _Installation_-Section of the linked page section.
    This includes: Installing Docker, Installing Visual Studio Code itself, and Installing the necessary Extension.

2.  Clone the project to your machine

3.  Open the folder of the repo in Visual Studio Code (_File - Open Folder…_).

4.  If you get a prompt at the lower right corner informing you about the presence of the devcontainer definition, you can press the accompanying button to enter it. **Otherwise,** you can also execute the Visual Studio Command `Remote-Containers: Open Folder in Container` yourself (_View - Command Palette_ → _type in the mentioned command_).

I recommend using `Remote-Containers: Rebuild Without Cache and Reopen in Container` once here and there as the devcontainer feature does have some problems recognizing changes made to its definition properly some times.

You may need to configure your host system to enable the container to use your SSH Keys.

The procedure is described [ in the official devcontainer docs under "Sharing Git credentials with your container"](https://code.visualstudio.com/docs/remote/containers#_sharing-git-credentials-with-your-container).

## 🍪 CookieCutter

This Project shall be kept in sync with [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role) using [cruft](https://github.com/cruft/cruft) (if possible) or manual alteration (if needed) to the best extend possible.

> ![Official Example Usage of `cruft update`](https://raw.githubusercontent.com/cruft/cruft/master/art/example_update.gif)

### 🕗 Changelog

When a new tag is pushed, an appropriate GitHub Release will be created by the Repository Maintainer to provide a proper human change log with a title and description.

## ℹ️ General Linting and Styling Conventions

General Linting and Styling Conventions are [**automatically** held up to Standards](https://stackoverflow.blog/2020/07/20/linters-arent-in-your-way-theyre-on-your-side/) by various [`pre-commit`](https://pre-commit.com/) hooks, at least to some extend.

Automatic Execution of pre-commit is done on each Contribution using [`pre-commit.ci`](https://pre-commit.ci/)[\*](#note_pre-commit-ci). Pull Requests even automatically get fixed by the same tool, at least by hooks that automatically alter files.

Not to confuse: Although some pre-commit hooks may be able to warn you about script-analyzed flaws in syntax or even code to some extend (for which reason pre-commit’s hooks are **part of** the test suite), pre-commit itself does not run any real Test Suites. For Information on Testing, see [🧪 Testing](#testing).

Nevertheless, I recommend you to integrate pre-commit into your local development workflow yourself.

This can be done by cd’ing into the directory of your cloned project and running `pre-commit install`. Doing so will make git run pre-commit checks on every commit you make, aborting the commit themselves if a hook alarm’ed.

You can also, for example, execute pre-commit’s hooks at any time by running `pre-commit run --all-files`.

# 💪 Contributing

![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square) [![Open in Visual Studio Code](https://img.shields.io/static/v1?logo=visualstudiocode&label=&message=Open%20in%20Visual%20Studio%20Code&labelColor=2c2c32&color=007acc&logoColor=007acc)](https://open.vscode.dev/JonasPammer/ansible-role-goaccess)

The following sections are generic in nature and are used to help new contributors. The actual "Development Documentation" of this project is found under [📝 Development](#development).

## 🤝 Preamble

First off, thank you for considering contributing to this Project.

Following these guidelines helps to communicate that you respect the time of the developers managing and developing this open source project. In return, they should reciprocate that respect in addressing your issue, assessing changes, and helping you finalize your pull requests.

## 🍪 CookieCutter

This Project owns many of its files to [the CookieCutter it was originally templated from](https://github.com/JonasPammer/cookiecutter-ansible-role).

Please check if the edit you have in mind is actually applicable to the template and if so make an appropriate change there instead. Your change may also be applicable partly to the template as well as partly to something specific to this project, in which case you would be creating multiple PRs.

## 💬 Conventional Commits

A casual contributor does not have to worry about following [_the spec_](https://github.com/JonasPammer/JonasPammer/blob/master/demystifying/conventional_commits.adoc) [_by definition_](https://www.conventionalcommits.org/en/v1.0.0/), as pull requests are being squash merged into one commit in the project. Only core contributors, i.e. those with rights to push to this project’s branches, must follow it (e.g. to allow for automatic version determination and changelog generation to work).

## 🚀 Getting Started

Contributions are made to this repo via Issues and Pull Requests (PRs). A few general guidelines that cover both:

- Search for existing Issues and PRs before creating your own.

- If you’ve never contributed before, see [ the first timer’s guide on Auth0’s blog](https://auth0.com/blog/a-first-timers-guide-to-an-open-source-project/) for resources and tips on how to get started.

### Issues

Issues should be used to report problems, request a new feature, or to discuss potential changes **before** a PR is created. When you [ create a new Issue](https://github.com/JonasPammer/ansible-role-goaccess/issues/new), a template will be loaded that will guide you through collecting and providing the information we need to investigate.

If you find an Issue that addresses the problem you’re having, please add your own reproduction information to the existing issue **rather than creating a new one**. Adding a [reaction](https://github.blog/2016-03-10-add-reactions-to-pull-requests-issues-and-comments/) can also help be indicating to our maintainers that a particular problem is affecting more than just the reporter.

### Pull Requests

PRs to this Project are always welcome and can be a quick way to get your fix or improvement slated for the next release. [In general](https://blog.ploeh.dk/2015/01/15/10-tips-for-better-pull-requests/), PRs should:

- Only fix/add the functionality in question **OR** address wide-spread whitespace/style issues, not both.

- Add unit or integration tests for fixed or changed functionality (if a test suite already exists).

- **Address a single concern**

- **Include documentation** in the repo

- Be accompanied by a complete Pull Request template (loaded automatically when a PR is created).

For changes that address core functionality or would require breaking changes (e.g. a major release), it’s best to open an Issue to discuss your proposal first.

In general, we follow the "fork-and-pull" Git workflow

1.  Fork the repository to your own Github account

2.  Clone the project to your machine

3.  Create a branch locally with a succinct but descriptive name

4.  Commit changes to the branch

5.  Following any formatting and testing guidelines specific to this repo

6.  Push changes to your fork

7.  Open a PR in our repository and follow the PR template so that we can efficiently review the changes.

# 🗒 Changelog

Please refer to the [Release Page of this Repository](https://github.com/JonasPammer/ansible-role-goaccess/releases) for a human changelog of the corresponding [Tags (Versions) of this Project](https://github.com/JonasPammer/ansible-role-goaccess/tags).

Note that this Project adheres to Semantic Versioning. Please report any accidental breaking changes of a minor version update.

# ⚖️ License

**[LICENSE](LICENSE)**

    MIT License

    Copyright (c) 2022, Jonas Pammer

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
