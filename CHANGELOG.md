#### 0.7.3 Important bug fix

- Fixed project-mode support for latest Node-RED versions.

#### 0.7.2 Important bug fix

- `envnodes` directory is optional.

#### 0.7.1 Fix for older Node-REDs

- Fixed missing flow-manager button for Node-RED versions below 1.0.0

#### 0.7.0 Flow handling enhancements

##### Features
- Remote Deployment to external Node-RED instances.
- envnodes supporting types .js .jsonata .json
- envnodes: can put as many files as the user wishes, no need to use filename "default".
- using js, you can "module.exports" the object immediately, or use a function to return the object. the function can be async.
- external flow loading on-demand All calls are on POST method, supporting {"action":"loadAll"/"reloadOnly"} and {"action":"removeOndemand"/"addOndemand"/"replaceOndemand", "flows: ["Flow 1","Flow 2"...]}
- Added RESTful calls (refer to README.md)

##### Improvements
- All RESTful calls protected with RED.auth.needsPermission (flows.write / flows.read)
- Documented all REST methods
- flow_visibility.json renamed to flow-manager-cfg.json
- added file flow-manager-nodes-order.json to keep nodes order, to remain in sync with UI nodes order, and avoid false-positives of "flows are not up to date" in UI.
- Changed filter-flows popup: compatible with all jQuery supported browsers (instead of chrome only), and flows selection is now "draggable" so you don't have to tick every flow you want separately.
- Improved flows state logic and representation: deployed/rev/mtime/hasUpdate/[oldRev/oldMtime] mtime means modified time of file.

##### Fixes
- fixed restriction of envnodes locked properties, and better popup description on UI when user is trying to modify them.

#### 0.6.0 Flow handling enhancements

- Performance improvements during deploys. 
- Support for envnode overriding using node name (not only by node id). 
- On Demand flow loading using RESTful requests (apart from filter-flows).
- Retrieving state of flows using RESTful requests (isDeployed/hasUpdate/onDemand).  

#### 0.5.2 Flow handling enhancements

- Performance improvement during initial boot.
- Fix for project mode when creating first project.

#### 0.5.1 Flow handling enhancements

- Flow renaming/removal triggered automatically by deploy.<br/>No need to delete/rename the flow in the file-system manually.
- Renamed "Show Flows" in filter popup to "Load Flows", better representing what it does.

#### 0.4.5 Bug Fix

- Ignoring non json/yaml files in flows directory

#### 0.4.4 Features & Improvements

- Replaced deprecated dependency fs-promise with fs-extra
- Project mode support
- Removed "Save Flow" button (triggered after each deploy)
- Removed dependency on deprecated library "asyncawait"

#### 0.4.3 YAML format

- Support for flow files in yaml format

#### 0.4.2 EnvNodes Visual Revert

- EnvNodes functionality improved<br/>Changes to envnode controlled properties are automatically reverted also on UI side.

#### 0.4.1 EnvNodes

- EnvNodes functionality improved<br/>Changes to envnode controlled properties are automatically reverted on NodeJS side.

#### 0.3.1: Maintenance

- Added missing keywords in package.json to appear in Node-RED library.

#### 0.3.0: First Release
