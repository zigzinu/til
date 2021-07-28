# How to insert arguments when running springboot debug mode with VSCode?

1. Click on settings icon on debug **Run and Debug** tab to open `launch.json`

![image](https://user-images.githubusercontent.com/83999058/127278392-1918e99e-9807-401a-844b-b09bee92492f.png)

2. Add `vmArgs` list to your **Application**'s configuration.

```
// launch.json
{
    // Use IntelliSense to learn about possible attributes.
    // Hover to view descriptions of existing attributes.
    // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Launch Current File",
            "request": "launch",
            "mainClass": "${file}"
        },
        {
            "type": "java",
            "name": "Launch CbecApplication",
            "request": "launch",
            "mainClass": "com.ktnet.cbec.CbecApplication",
            "projectName": "cbec",
            // "args": ["-Djasypt.encryptor.password=ktnetadmin1!"],
            "vmArgs": [
                "-Djasypt.encryptor.password=mypassword1!"
            ]
        }
    ]
}
```

3. **Start Debugging** by chosing the right configuration.

![image](https://user-images.githubusercontent.com/83999058/127278195-5ec14e4b-177a-4f4d-8ad3-28e29689450b.png)
