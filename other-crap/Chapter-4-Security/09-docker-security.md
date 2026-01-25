# Docker Security

* Containers and host share the same kernel
* When docker is running on the host, there are several docker processes running on the host.
* The hosts have their own namespace and containers have their own namespace.
* A container running can see only processes on its own namespace, it cannot say anything in the host's namespace.
* docker runs processes inside a container are usually run as a root user by default.
* If docker is to be instructed to run a container with a user other than the root user then use the --user command
* Docker processes by default do not have access to any processes running on the host. If this functionality is to be overridden
  the following arguments can be passed when running the container.



```shell
# Add a privilege
docker run --cap-add <option>

# Drop a privilege
docker run --cap-drop <option>

# Run with all privileged options
docker run --privileged # runs with all options.
```
