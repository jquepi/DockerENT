<p align="center">
  <img width="300" height="300" src="https://raw.githubusercontent.com/r0hi7/DockerENT/master/static/logo.png">
  <br>
  <img alt="GitHub Workflow Status" src="https://img.shields.io/github/workflow/status/r0hi7/DockerENT/Upload%20Python%20Package">
  <img alt="GitHub release (latest by date)" src="https://img.shields.io/github/v/release/r0hi7/DockerENT">
  <img alt="PyPI - Downloads" src="https://img.shields.io/pypi/dm/DockerENT">
  <img alt="Libraries.io dependency status for GitHub repo" src="https://img.shields.io/librariesio/github/r0hi7/DockerENT">
  <img alt="GitHub code size in bytes" src="https://img.shields.io/github/languages/code-size/r0hi7/DockerENT">
  <img alt="GitHub" src="https://img.shields.io/github/license/r0hi7/DockerENT">
  <br>
  <img alt="GitHub issues" src="https://img.shields.io/github/issues-raw/r0hi7/DockerENT">
  <img alt="GitHub stars" src="https://img.shields.io/github/stars/r0hi7/DockerENT">
  <img alt="Twitter Follow" src="https://img.shields.io/twitter/follow/sec_r0?style=social">
  <img alt="GitHub followers" src="https://img.shields.io/github/followers/r0hi7?style=social">
  <br>
  <br>
  <a target="_blank" href="https://twitter.com/intent/tweet?text=DockerENT: Docker runtime security scanning tool. https://github.com/r0hi7/DockerENT" title="Share on Twitter"><img src="https://img.shields.io/twitter/url/http/shields.io.svg?style=social&label=Share%20on%20Twitter"></a>
  
</p>

# DockerENT

DockerENT is activ**E** ru**N**time application security scanning **T**ool (**RAST** tool). It is a pluggable framework written in Python. It comes with a **CLI application** and a clean **Web Interface** written using [StreamLit](https://www.streamlit.io/). 

DockerENT has been designed to detect weak security misconfigurations in production deployments which can lead to severe consequences. This application connects with running containers in the system and fetches the list of weak and vulnerable runtime configurations and generates a report.
If invoked through **web** interface, it can display the scan and audit report in the UI itself.

## How to Run

### TL;DR
In hurry to test this? Download the latest **stable REL** from PyPi and run the Web App, everything else is intuitive.

`pip install DockerENT`

Then run the application like: 

`DockerENT -w`

Thats it.

### Run the latest master
DockerENT has been designed to keep simplicity and usability in mind. Currently you just have to clone the repository and download dependencies or build the Dockerfile. Once the dependencies are installed in local system we are good to run the tool and analyse the runtime configurations for running containers.

```bash
# Download and setup
git clone https://github.com/r0hi7/DockerENT.git
cd DockerENT
make venv
source venv/bin/activate

# Run
python -m DockerENT --help 
usage: Find the vulnerabilities hidden in your running container(s).
       [-h] [-d [DOCKER_CONTAINER]] [-p [DOCKER_PLUGINS]]
       [-d-nw [DOCKER_NETWORK]] [-p-nw [DOCKER_NW_PLUGINS]] [-w]
       [-n [PROCESS_COUNT]] [-a] [-o [OUTPUT]]

optional arguments:
  -h, --help            show this help message and exit
  -w, --web-app         Run DockerENT in WebApp mode. If this parameter is
                        enabled, other command line flags will be ignored.
  -n [PROCESS_COUNT], --process [PROCESS_COUNT]
                        Run scans in parallel (Process pool count).
  -a, --audit           Flag to check weather to audit results or not.

  -d [DOCKER_CONTAINER], --docker [DOCKER_CONTAINER]
                        Run scan against the running container.
  -p [DOCKER_PLUGINS], --plugins [DOCKER_PLUGINS]
                        Run scan with only specified plugins.
  -p-nw [DOCKER_NW_PLUGINS], --nw-plugins [DOCKER_NW_PLUGINS]
                        Run scan with only specified plugins.

  -d-nw [DOCKER_NETWORK], --docker-network [DOCKER_NETWORK]
                        Run scan against running docker-network.

  -o [OUTPUT], --output [OUTPUT]
                        Output plugin to write data to.

# or via the container
docker build . -t dockerent
docker run -d -v /var/run/docker.sock:/var/run/docker.sock -p 8501:8501 --name dockerent dockerent
# Then just open your browser to http://localhost:8051
```
See this quick video to get started with.
<p align="center">
    <a href="https://asciinema.org/a/Zmtk8KjlbPr1vHa3efZiCIDs2" target="_blank">
        <img src="https://asciinema.org/a/Zmtk8KjlbPr1vHa3efZiCIDs2.svg" />
    </a> 
</p>    

### Features
- Plugin driven framework.
- Use low level docker api to interact with running containers.
- Clean and Easy to Use UI.
- Comes with **9 docker scan plugins** out of which, **6 plugins** can audit results.
    - Entire list : [Docker Scan Plugins](https://github.com/r0hi7/DockerENT/tree/master/DockerENT/docker_plugins)
- Framework ready to work docker-networks.
    - Entire list : [Docker Network Scan plugins](https://github.com/r0hi7/DockerENT/tree/master/DockerENT/docker_nw_plugins)
- [Output plugins](https://github.com/r0hi7/DockerENT/tree/master/DockerENT/output_plugins) can write to `file` and `html` sinks.
- The only open source interactive docker scanning tool.
- Can run plugins in parallel.
- Under active development :smile:.

### How to Create your own Plugin.
- Have some **idea** to perform runtime scan.
- Copy the same file to create your `demo` plugin.
```bash 
cp DockerENT/docker_plugins/docker_sample_plugin.py DockerENT/docker_plugins/docker_demo_plugin.py
```
- Just make sure, you maintain following structure.
```python
_plugin_name = 'demo_plugin'

def scan(container, output_queue, audit=False, audit_queue=None):
    _log.info('Staring {} Plugin ...'.format(_plugin_name_))

    res = {}

    result = {
        'test_class': {
            'TEST_NAME': ['good']
        }
    }

    res[container.short_id] = {
        _plugin_name_: result
    }
    
    # Do something magical.

    _log.info('Completed execution of {} Plugin.'.format(_plugin_name_))

    '''Make Sure you put dict of following structure in Q.
    {
        'contiainer_id': {
            'plugin_name': {
                'test_name_demo1': {
                    resultss:[]
                },
                'test_name_demo2': {
                    results: []
                }
            }
        }
    }
    '''
    output_queue.put(res)

    if audit:
        _audit(container, res, audit_queue)

def _audit(container, results, audit_queue):
    '''Make Sure to add dict of following structure to Audit Q
    res = {
        "container_id": [
            "_plugin_name_, WARN/INFO/ERROR, details"
        ]
    }
    '''
    # Magical logic to perform Audit.
    audit_queue.put(res)
```
- Thats it. Still confused, Explain me the idea in [Issues](https://github.com/r0hi7/DockerENT/issues) and will review and help you out, or we may end up working on it together.
- This plugin will automatically come to drop down in UI. :smile: Easy right.
- Sit back and eval results.

### Plugins Features:
| Plugin Name        	| Plugin File                                                        	| Feature                              	| Audit                               	|
|--------------------	|--------------------------------------------------------------------	|--------------------------------------	|-------------------------------------	|
| CMD_HISTORY        	| [File](DockerENT/docker_plugins/docker_cmd_history_info.py)        	| Identify shell history               	| Root history and User shell history 	|
| FILESYSTEM         	| [File](DockerENT/docker_plugins/docker_filesystem_info.py)         	| Identify RW File Systems             	| If RW file systems are present.     	|
| NETWORK            	| [File](DockerENT/docker_plugins/docker_network_info.py)            	| Identify Network state               	| Identifies All mapped ports.        	|
| PLAINTEST_PASSWORD 	| [File](DockerENT/docker_plugins/docker_plaintext_password_info.py) 	| Identify password in different files 	|                                     	|
| SECURITY_PROFILES  	| [File](DockerENT/docker_plugins/docker_security_profiles_info.py)  	| Identify Weak Security Profiles      	| List Weak security profiles.        	|
|USER_INFO      |         [File](DockerENT/docker_plugins/docker_user_info.py)|Identify user info| List permissions in passwd and other sensitive files|
|SYSTEM_INFO      |       [File](DockerENT/docker_plugins/docker_system_info.py)|Identify docker system info| No Audit|
|FILES_INFO|[File](DockerENT/docker_plugins/docker_file_info.py)|Identify world writeable directories and files|List all such files.|
|PROC_INFO| [File](DockerENT/docker_plugins/docker_process_info.py)| Identify the list of process in docker system| |

### CLI interface
#### Pros
- [Rich](https://github.com/willmcgugan/rich) Logging interface, can help in easy debugging through extensive debug logs.
- Can run in parallel, just pass `-n <count>`, to specify the processors in parallel.
- Can dump output in `JSON` and `HTML` file.

#### Cons
- Audit output is not dumped to file.
- Selecting multiple specific dockers is pain.

### UI Interface
#### Pros
- **Clean**, and easy to use UI.
- Everything at one single page.
- Ease of selecting **multilpe** docker images, **multilpe** plugins and **multilpe** docker-networks.
- Audit report **present**.

#### Cons
- Logging interface not Rich.
- `JSON` reports are bulky.
- Rely on third party lib [StreamLit](https://www.streamlit.io/), all issues with framework are inherent.


### Help Make this tool better
- Create a PR, Issues are more than welcome.
- Try it, test it and enhance it.
