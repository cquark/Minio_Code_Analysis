## callstack
main@main.go  
\> [Main@main.go@cmd](https://github.com/minio/minio/blob/712fe1a8dfd64bbc3fde1863cbbf55ae17e7c911/main.go#L30)  
\> [cli.App.Run](https://github.com/minio/minio/blob/712fe1a8dfd64bbc3fde1863cbbf55ae17e7c911/cmd/main.go#L225)  
\> [HandleAction](https://github.com/minio/cli/blob/bc9585b3b22dad2a21fd11d5389bd3f3e402f7a7/app.go#L269)  
\> [serverCmd](https://github.com/minio/cli/blob/bc9585b3b22dad2a21fd11d5389bd3f3e402f7a7/app.go#L488)  
\> [serverMain](https://github.com/minio/minio/blob/6d18dba9a20d6e925c1792648fa8b2702d71b5f9/cmd/server-main.go#L744)  
 

## bootstrap process @ [serverMain](https://github.com/minio/minio/blob/6d18dba9a20d6e925c1792648fa8b2702d71b5f9/cmd/server-main.go#L744)  
1. Initialize globalConsoleSys system
2. Load ENV variables from files
3. Handle early server environment vars
4. Handle all server command args and build the disks layout
5. DNS cache subsystem to reduce outgoing DNS requests
6. Handle all server environment vars.
7. Load the root credentials from the shell environment or from the config file if not defined, set the default one.
8. Perform any self-tests
9. Initialize KMS configuration
10. Initialize all help
11. Initialize all sub-systems
12. Is distributed setup, error out if no certificates are found for HTTPS endpoints.
13. Check for updates in non-blocking manner.
14. Set system resources to maximum.
15. Verify kernel release and version.
16. Initialize grid
17. Initialize lock grid
18. Configure server.  
    <ol type="a">
        <li>configureServer</li>
	    <li>verifying system configuration</li>
	    <li>newObjectLayer</li>
	    <li>waitForQuorum</li>
	    <li>initServerConfig</li>
    </ol>
19. Initialize users credentials and policies in background right after config has initialized.  
    <ol type="a">
	    <li>globalIAMSys.Init</li>
	    <li>Initialize Console UI</li>
	    <li>if we see FTP args, start FTP if possible</li>
	    <li>If we see SFTP args, start SFTP if possible</li>
    </ol>
20. asynchronous inits  
    <ol type="a">
	    <li>Initialize data scanner.</li>
	    <li>Initialize background replication</li>
	    <li>Initialize background ILM worker poool</li>
	    <li>Initialize transition tier configuration manager</li>
	    <li>Initialize bucket notification system.</li>
	    <li>List buckets to initialize bucket metadata sub-sys.</li>
	    <li>Initialize bucket metadata sub-system.</li>
	    <li>initialize replication resync state.</li>
	    <li>Initialize site replication manager after bucket metadata</li>
	    <li>Populate existing buckets to the etcd backend</li>
	    <li>Initialize batch job pool.</li>
	    <li>Prints the formatted startup message, if err is not nil then it prints additional information as well.</li>
	    <li>Print a warning at the end of the startup banner so it is more noticeable</li>
    </ol>
21. globalMinioClient
22. startResourceMetricsCollection
23. Add User-Agent to differentiate the requests.
