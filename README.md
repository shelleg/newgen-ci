**CI/CD Infrastructure Automation**
================================

An Infra automation code which provides several software integration and 3rd party automation based on
Ansible playbooks and community-developed roles.

**REQUIREMENTS**
================================
Java 1.8.X

**FEATURES**
================================
* **Nexus** - Binary repository (apt, .hpi, docker-registry, caching)
* **Juseppe** - Jenkins custom onpremise update-Center for customized plugin sets
* **Jenkins** - customed Jenkins setup based on Vanilla Jenkins (v2.32) Offline installation supported via Nexus

**Variables & Defaults**
================================
* `seed_sanjer: ''` - **MUST** be filled with the correct 'sanjer' IP 
* `cj_http_port: 8080` - https support when added will be done via nginx / apache2 
* `cj_channel: latest-LTS` - this controls what repository will the rpm/qpt be taken from see the `vars/{{ ansible_os_family }}.yml` to see the urls.
* `cj_version: '23.2'` - this controls what version to use -- **please note this overrides the latest v.s latest-LTS ...** considering the download base url for Latest & LTS is the same ...
* `cj_application_context: ''` - defaults to none which is really `http://jenkins-host/` change it to your liking ...
*  Jenkins directories used thought the role

** Triggers ** - Groupvars
* `cj_juseppe: true` - Juseppe integration with Jenkins
* `cj_install_online_mode` - Off/Online installion flow
* `cj_nexus: true` - Nexus integration with Juseppe/Jenkins
* `cj_seed_demo: true` - Runs the default job 'seed' post-installation

```yamlex
    cj_home: /var/lib/jenkins                          # the default home directory just like the apt/rpm
    cj_log_dir: /var/log/jenkins                       # default log file
    cj_cache_dir: /var/cache/jenkins                   # cache directory { used to focus on jenkins-cli.jar
    cj_init_dir: "{{ cj_home }}/init.groovy.d"    # post installation scripts go here 
    cj_update_dir: "{{ cj_home }}/updates"        # unpadte.center.json goes here
    cj_webroot: /var/cache/jenkins/war                 # 
    cj_plugins_tmp_folder: /tmp/plugins                # will probebly be removed in next release { still figuring out offline instllation }
```
*  Set jenkins configuration [ I used the [squid proxy](https://github.com/shelleg/ansible-role-squid) role]
```yamlex
    cj_set_proxy: false
    cj_proxy_host: "{{ proxy_server }}"
    cj_proxy_port: "{{ proxy_port |d('3148') }}"
    #cj_no_proxy_hosts: will be set in tasks
    cj_proxy_conf_file: "{{ cj_home }}/proxy.xml"
    # in case we want to test jenkins without a proxy when cj_set_proxy is set to true
    cj_remove_proxy: false
```
* Disable the startup wizard (I will install these plugins anyway later on) 
```yamlex
    cj_run_startup_wizard: "false" # String becuase the jvm arg becomes False ...
    cj_jvm_args: "-Djava.awt.headless=true -Djenkins.install.runSetupWizard={{ cj_run_startup_wizard }}"
    cj_cmd_args: "--webroot={{ cj_webroot }} --httpPort={{ cj_http_port }}"
```
* DEPRECATED / USE AT OWN RISK - running jenkins as a different user, this will probebly still work so I did not throw away a couple of hours of work ... 
```yamlex
    # The system level user Jenkins should runas
    cj_runas: false
    #cj_custom_user: user
    #cj_custom_group: user
    cj_runas_user: "{{ cj_custom_user | d('jenkins') }}"
    cj_runas_group: "{{ cj_custom_group | d('jenkins') }}"
    cj_runas_sudoer: true
```
* Authorization strategy basic - used to populate / generate the `templates/basic-security.groovy.j2` file inserted into `init.groovy.d` directory
```yamlex
    cj_allow_anon_read: true
    cj_basic_auth: true
    cj_local_admin_users:
      - { username: admin, password: admin }
      - { username: jjb, password: jjb }
    
    cj_local_regular_users:
      - { username: user1, password: user1 }
      - { username: user2, password: user2 }
```

* jenkins-cli related - most plugin ops are done via CLI so basically you can perfrom these remotely or via Docker running jenkins (same goes for groovy scripts metioned above)
```yamlex
    cj_cli_jar_location: "{{ cj_cache_dir }}/war/WEB-INF/jenkins-cli.jar"`
    cj_cli_hostname: 'localhost'
```

* Jenkins material theme
```yamlex
    # could be: red, pink, purple, deep-purple see full list @ -> http://afonsof.com/jenkins-material-theme/
    cj_theme_color: amber
    cj_theme_conf_file: org.codefirst.SimpleThemeDecorator.xml
    cj_theme_css_url: https://jenkins-contrib-themes.github.io/jenkins-material-theme/dist/material-{{ cj_theme_color }}.css
    cj_theme_js_url: https://cdn.rawgit.com/djonsson/jenkins-atlassian-theme/gh-pages/theme.js

```
* Jenkins plugin sets ... this still needs some work, this is the begining which will change based on requirements
```yamlex
    # choose plugin sets to use
    cj_wizard: true
    cj_theme: true
    cj_setup_matrix_auth_strategy: true
    cj_setup_dsl_plugins: true
    cj_goodies: true
```
* Jenkins short see job demo:
```yamlex
  cj_seed_demo: true # this will probebly be set to false upon release
```
* Jenkins plugin lists - this is also a preparation for offline installation mode - as an example:
```yamlex
    cj_wizard_plugins:
      - { name: "bouncycastle-api", version: "2.16.0", enabled: "true", extention: "hpi" }
      - { name: "cloudbees-folder", version: "5.15", enabled: "true", extention: "hpi" }
      - ...
    cj_goodie_plugins:
      - { name: "greenballs" }
```

* Juseppe role variables
```yamlex
    juseppe_container_hostname: juseppe
    juseppe_container_name: lanwen/juseppe
    juseppe_container_tag: 1.0.0
    juseppe_container_port: 8080
    juseppe_host_port: 9090
    juseppe_docker_host_ip: localhost
    juseppe_cert_dir: /var/lib/juseppe
    juseppe_plugin_cache_dir: /var/cache/juseppe
```
**HOW TO**
======================================================
* Clone this git repo to /opt `git clone git@github.com:Chrislevi/newgen-ci.git /opt`
* Run `ansible-galaxy install -r requirements.yml`
* Configure setup via `playbooks/group_vars/all/{00_general,10_plugins}`
* Place your ssh key into your GitHub account
* run `ansible-playbook playbooks/default.yml`
* Browse localhost:8080 , Login! WOALLA! 
