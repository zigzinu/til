# How to open HTML file on my browser when VSCode is connected to SSH?

## Goal 

Open server's (ex. Ubuntu VM) **.html** file on host's (ex. Windows) browser.

## Problem

![image](https://user-images.githubusercontent.com/83999058/127288047-e63714f5-8ad4-4d43-937f-d2bad7b10365.png)

**Open in browser** extension allows you to open a **.html** file with browser. 
However, if your VSCode is connected to a server through SSH, the browser doesn't understand server's directory.

![image](https://user-images.githubusercontent.com/83999058/127288381-ac74dcda-8b8f-4616-868e-7c581b94bd17.png)

![image](https://user-images.githubusercontent.com/83999058/127288321-4d0d282f-51f1-4536-ad49-ab5653b63d6c.png)

## Possible solution

Utilize python to allow host (Windows) an access to server's directory.

```
python3 -m http.server
```

![image](https://user-images.githubusercontent.com/83999058/127416305-89a3a28c-c09d-4852-853e-3ef0a4394950.png)

Now python has hosted a server that allows access to current directory where the command has executed.

![image](https://user-images.githubusercontent.com/83999058/127416435-b29e66e1-615c-48ed-8cd0-ed4074c9237a.png)

You can open what files in the directory.

![image](https://user-images.githubusercontent.com/83999058/127416473-c0473271-4374-4e79-ac23-b1fda135a846.png)

😃
