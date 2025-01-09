## _Flow Manager_ module for node-red

Flow Manager separates your flow json to multiple files

[View change log](https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/blob/master/CHANGELOG.md)

### Installation

This assumes you have [Node-RED](https://nodered.org) already installed and working, if you need to install Node-RED see [here](https://nodered.org/docs/getting-started/installation)

**NOTE:** This requires [Node.js](https://nodejs.org) v14+

Install via Node-RED Manage Palette

```
node-red-contrib-flow-manager
```

Install via npm

```shell
$ cd ~/.node-red
$ npm install node-red-contrib-flow-manager
# Restart node-red
```

### Usage 
After installation of this module, during first boot only, A migration process will initiate.

During migration, your main flow json file will be split to multiple files which will store your node information.<br/>
It is advised to **back up your main flow json**, before running this module for the first time.

Node-RED startup process after migration:<br/>
All of your flow files are combined into a single JSON object, which is then loaded and served as your main flow object.

For that reason, it is advised to add your "fat" flow json file to .gitignore because from that moment, the flows are saved separately.

The nodes will be stored in the following subdirectories of your Node-RED path as such:
* `/flows/`**`flow name`**
* `/subflows/`**`subflow name`**
* `/config-nodes.json` (global config nodes will be stored here)

It's a good idea to add these paths to your version control system. 

#### ![Filter Flows](https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/raw/0.7.1/img/filter_flows_button.png)
* Allows selecting which flows Node-RED will load, and which will be ignored and not loaded, **not only in Node-RED's UI, also in it's NodeJS process.** <br/>
* Unselected flows are NOT deleted, only "ignored" until you select them again using `Filter Flows`.
* Filtering configuration is stored in `flow-manager-cfg.json` file under your Node-RED path.
* if `flow-manager-cfg.json` file does not exist, or exists but malformed, or contains an empty JSON array, then all flows will be loaded and no filtering is done.
* ![Filter Flows Popup](https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/raw/0.7.1/img/filter_flows_popup.png)
    
### envnodes
envnodes allows configuration of nodes using an external source and custom logic.<br/>
example:
create "envnodes/someName.jsonata" in your Node-RED project directory and put these contents:

```json5
  (
     $config := require(basePath & "/someConfig.json");
     {
       /* mqtt config node */
       "21bcf36a.891e4d": {
          "broker": $config.mqtt.broker,
          "port": $config.mqtt.port
       },
       "name:MyInject": {
          "topic": "niceTopic"
       }
     };
  )
```
The result would be that your mqtt config node will use values from an external configuration file (which is useful in some cases),
And a topic value will be inserted to an inject node matched with the given name.

Note `basePath` is the path to your Node-RED directory.<br/>
When using Node-RED's "project mode", the value is the project folder path.

Supported envnode file ext: .jsonata .js .json
when using .js you can either module.exports = {...} the object directly, or use a function/async function (for example to read external file) and return an object.<br/>
js example:

```javascript
const fs = require('fs-extra');

module.exports = async function() {
    const configTopic = await fs.readJson('./config').topic;
    return {
        "name:MyInject": {
            "topic": configTopic
        },
        "cff2a203.8158a": {
            "someProperty": "myProperty"
        }
    }
}

```

Attempting to change any envnode controlled property via Node-RED UI/AdminAPI will be cancelled (with a warning popup) to keep original values defined in your envnodes configuration.

![EnvNodes Warning](https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/raw/0.7.1/img/envnodes_warning.png)

### Remote Deploy
![Remote Deploy Button](https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/raw/0.7.1/img/remote_deploy_button.png)

If you work in a server-client environment, where you work locally on your flows, and once in a while need to push your local changes to a remote process running Node-RED.

Do the following to enable Remote Deployment feature:<br/>
* Add remote definitions to `flow-manager-cfg.json` (you can add as many as you want, choose any name you wish for each remote)
    -   ```json
        {
          "filter": [],
          "fileFormat": "json",
          "remoteDeploy": {
            "remotes": [
              {
                "name": "Production",
                "nrAddress": "http://yourAddress:1881"
              },
              {
                "name": "Staging",
                "nrAddress": "http://yourOtherAddress:1881"
              }
            ]
          }
        }
        ``` 
* Restart Node-RED
* You should see the new "Remote Deploy" button on top of the Node-RED UI.

![Remote Deploy Remote Selection](https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/raw/0.7.1/img/remote_deploy_select_remote.png)

![Remote Deploy Diff Tool](https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/raw/0.7.1/img/remote_diff.png)

    
### YAML flow file format
You can configure flow-manager to load/save flow files in YAML format instead oj JSON, by modifying file `flow-manager-cfg.json` as such:

```json
{
  ...
  "fileFormat": "yaml"
  ...
}
```
The advantage is the code within function node becomes easier to read when inspecting the flow file itself.<br/>
See comparison below
#### YAML
```yaml
- id: 493f0b3.b3b48f4
  type: function
  z: 70dd6be0.e35274
  name: 'SomeFuncNode'
  func: |-
      const str = "hello";
      console.log(str + ' world');
      msg.payload = 'test';
      return msg;
  outputs: 1
  noerr: 0
  x: 580
  'y': 40
  wires:
    - []
```
#### JSON
```json
{
    "id": "493f0b3.b3b48f4",
    "type": "function",
    "z": "70dd6be0.e35274",
    "name": "SomeFuncNode",
    "func": "const str = \"hello\";\nconsole.log(str + ' world');\nmsg.payload = 'test';\nreturn msg;",
    "outputs": 1,
    "noerr": 0,
    "x": 580,
    "y": 40,
    "wires": [
        []
    ]
}
```
  
  
### flow-manager RESTful API

#### Get all flow names

```shell
curl --request GET \
  http://localhost:1880/flow-manager/flow-names
```

Response:

```json
  ["Flow 1.json","Flow 2.json","Flow 3.json","Flow 4.json"]
```

#### Get flow-manager-cfg.json contents

```shell
curl --request GET \
  http://localhost:1880/flow-manager/cfg
```

#### get / change filter flows
get

```shell
curl --request GET \
  http://localhost:1880/flow-manager/filter-flows
```

change

```shell
curl --header "Content-Type: application/json" \
  --request PUT \
  --data '["Flow 1", "Flow 2"]' \
  http://localhost:1880/flow-manager/filter-flows
```

#### Modify flow files (add, update, delete any flow/subflow files)

Request URL template:

```
http://localhost:1880/flow-manager/flow-files/:type/:fileName?
```
`type` can be `flow`/`subflow`/`global`<br/>
can pass plural too: `flows`/`subflows`

Notice `fileName` is optional if you request "global" flow, leave out the `fileName` url part.

get

```shell
curl --request GET \
  http://localhost:1880/flow-manager/flow-files/flow/MyFlow
```

delete

```shell
curl --request DELETE \
  http://localhost:1880/flow-manager/flow-files/flow/MyFlow
```

add/update (mtime/atime query params are optional, in case you wish to set mtime "Modified Time" or atime "Access Time" after the file is written)

```shell
curl --header "Content-Type: application/json" \
  --request POST \
  --data '[{"id":"d4366369.5303d","type":"tab","label":"NewFlow","disabled":false,"info":""}]' \
  http://localhost:1880/flow-manager/flow-files/flow/NewFlow
```

Example with mtime query param:

```shell
curl --header "Content-Type: application/json" \
  --request POST \
  --data '[{"id":"d4366369.5303d","type":"tab","label":"NewFlow","disabled":false,"info":""}]' \
  'http://localhost:1880/flow-manager/flow-files/flows/NewFlow?mtime=8/16/2020,%206:12:17%20PM'
```

#### On Demand flow loading using external RESTful requests
During runtime, you can request flow-manager to load any flow file placed in the flows directory,<br/>
by sending a POST request with the flows you wish to load (regardless of whether you configured any filter-flows or not)

Request to deploy all flows / redeploy current flows:

```shell
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"action":"loadAll"/"reloadOnly" }' \
  http://localhost:1880/flow-manager/states
```

Request to add/remove/replace any flow:

```shell
curl --header "Content-Type: application/json" \
  --request POST \
  --data '{"action":"addOndemand"/"removeOndemand"/"replaceOndemand" "flows":["Flow 1","Flow 2"] }' \
  http://localhost:1880/flow-manager/states
```

#### Retrieve flow states for all flows, subflows, and global nodes (config nodes)

Retrieve state for specific type and flow.<br/>
`http://localhost:1880/flow-manager/states/:type/:flowName`

Retrieve state for all flows of a given type.<br/>
`http://localhost:1880/flow-manager/states/:type`

Retrieve state for all flows/subflows/global.<br/>
`http://localhost:1880/flow-manager/states/`

`:type` can be subflow/flow/global (if `global`, there's no need for `:flowName`).

Example response for specific flow (`http://localhost:1880/flow-manager/states/flow/Flow1`)

```json
{
    "deployed": true,
    "hasUpdate": true,
    "onDemand": false,
    "rev": "477a9952b50e161afdf2e4bb6b84ee31",
    "mtime": "2020-08-16T13:54:23.000Z",
    "oldRev": "54d163c840657920aaac705dadecae2b", // If hasUpdate
    "oldMtime": "2020-08-14T17:30:10.000Z" // If hasUpdate
}
```
* `deployed` whether that flow file is deployed. 
* `hasUpdate` whether that flow file was changed externally, and can be deployed again to update.
* `onDemand` whether that flow was deployed by an OnDemand request, and is not part of filtered flows selection. 
* `rev/oldRev` revision (checksum) of file.
* `mtime/oldMtime` Modified Time of file. 

#### Access flow-manager RESTful API on remotes defined in `flow-manager-cfg.json`

Request template:

```
http://localhost:1880/flow-manager/remotes/:remoteName/[append any rest endpoint here]
```

Example using `flow-names`

```shell
curl --request GET \
  http://localhost:1880/flow-manager/remotes/Production/flow-names

```


[Change Log]: https://gitlab.com/monogoto.io/node-red-contrib-flow-manager/-/raw/0.7.1/CHANGELOG.md