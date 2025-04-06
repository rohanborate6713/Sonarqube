# SonarQube Troubleshooting

### Issue 1-->  Process exited with exit value [es]: 137
solution -->

- What Does Exit Code 137 Mean?
Exit code 137 typically means "Killed", usually due to the Linux OOM Killer (Out Of Memory). This happens when your system doesn’t have enough memory to run Elasticsearch, so the kernel kills it to save RAM.
- check available memory 
```
df -h
```
- Elasticsearch requires at least 2 GB of free memory to start reliably.
- Want to Check if OOM Killed It? (out of memory
```
dmesg | grep -i kill
```
- If you see something like “Out of memory: Kill process xxxx (java)...”, then that’s the reason.


### Issue 2-->  Process exited with exit value [es]: 143


• This means: "terminated by SIGTERM" — likely SonarQube shut it down because the web process failed.

- What This Means:
Your web server process crashed right after it started. Elasticsearch was fine, but SonarQube can’t run without the web service, so it shuts down everything.
- check for web logs 
```
cat opt/sonar/logs/web.log
```
- Common Causes for Web Process Failure
- 
![image.png](https://eraser.imgix.net/workspaces/ILx8Qc9xUVSJusyY9oJt/yuIy5hbLwHZ10ovGIULZ9qCXT8E3/dD3xtPuYnSPOol4Vg1zP9.png?ixlib=js-3.7.0 "image.png")

- check 
 /opt/sonar/conf/sonar.properties

```
sonar.web.host=0.0.0.0
sonar.web.port=9000
```
- then restart sonar qube
```
/opt/sonar/bin/linux-x86-64/sonar.sh restart
```


