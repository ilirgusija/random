# How to get set up on the Apple "Supercomputer"
## 0. Key Info!
This guide uses a lot of placeholder values so make sure you put in the correct values when you try this all yourself.
`[hostname or ip address]=130.15.101.67`
`[username]=user1 or user2 or ... or user6`
If you run into any issues try ChatGPT/Google/Stack Exchange first and if things still don't work then contact:  [Rohan](apple.bed@engsoc.queensu.ca) or [Kalena](apple.pres@engsoc.queensu.ca) or [Ilir](ilir.gusija@queensu.ca).

## 1. Installing Queen's VPN
# NOTE YOU NEED TO SET UP THE VPN ON QUEEN'S WIFI WHEN USING IT FOR THE FIRST TIME
If you know someone who already has it set up, you can get them to connect to vpn and have them hotspot you to emulate that

In order to be able to connect to the machine off-campus you will need a Queen's VPN set-up in order to connect. Otherwise you won't be able to connect. Use this link to set everything up
[End User - Connecting to Campus VPN using FortiClient on Windows and macOS (service-now.com)](https://queensu.service-now.com/esm?id=kb_article&sysparm_article=KB0012696&sys_kb_id=b5eaa2b587bbb55064edf29acebb3559)

## 2. Installing SSH
### For Windows

1.  **Windows 10 and Later (Built-in SSH Client)**
    
    -   Windows 10 and later versions come with a built-in SSH client, so you might not need to install anything.
    -   To check if it's installed, open Command Prompt or PowerShell and type `ssh`. If it's installed, you'll see the usage instructions.
    -   If not installed, you can enable it by going to `Settings` -> `Apps` -> `Optional Features` -> `Add a feature`, and then select "OpenSSH Client" and click "Install".

### For macOS

1.  **macOS (Built-in SSH Client)**
    -   macOS comes with a built-in SSH client.
    -   Open Terminal.
    -   You can start an SSH session by typing `ssh [username]@[hostname or IP address]`.

### For Linux

1.  **Linux (Most Distros Have SSH Client Installed)**
    
    -   Most Linux distributions come with an SSH client installed by default.
    -   Open a terminal.
    -   Connect to a server by typing `ssh [username]@[hostname or IP address]`.
    -   For example, `ssh user@example.com`.
2.  **Installing SSH Client Manually**
    
    -   On Debian/Ubuntu-based distros: `sudo apt-get install openssh-client`.
    -   On Fedora/RHEL-based distros: `sudo dnf install openssh-clients`.
    -   On Arch Linux: `sudo pacman -S openssh`.

## 3. (Optional) Setting up SSH Keys for passwordless logins

If you don't want to have to type in the password for your user on the supercomputer every time you log in then you can do this!
### For Windows

1.  **Using Built-in OpenSSH Client (Windows 10 and Later)**
    
    -   Open PowerShell or Command Prompt.
    -   Generate a new SSH key pair with `ssh-keygen`.
    -   Follow the prompts to specify the file location and passphrase (optional but recommended for additional security).
    -   Your SSH key pair will be saved in the `.ssh` directory inside your user's home folder (`C:\Users\YourUsername\.ssh\` by default).
2.  **Adding the SSH Key to the SSH Agent**
    
    -   Start the ssh-agent in the background using `Start-Service ssh-agent`.
    -   Add your private key to the ssh-agent using `ssh-add ~\.ssh\id_rsa` (assuming your private key is named `id_rsa`).
3.  **Copy the Public Key to the Remote Server**
    
    -   Display and copy your public key with `Get-Content ~/.ssh/id_rsa.pub`.
    -   Paste this key into the `~/.ssh/authorized_keys` file on your remote server.
    - ^ Administrator might need to run something to make this part work so if you still get prompted to enter your password you might need to contact Rohan or Kalena to do that for you.

### For macOS & Linux

1.  **Generating the SSH Key Pair**
    
    -   Open the Terminal.
    -   Create a new SSH key pair with `ssh-keygen`.
    -   Follow the prompts to specify the file location and passphrase.
    -   The keys will be stored in `~/.ssh/`.
2.  **Adding the Key to the SSH Agent**
    
    -   Start the ssh-agent using `eval "$(ssh-agent -s)"`.
    -   Add your private key to the ssh-agent with `ssh-add -K ~/.ssh/id_rsa`.
3.  **Copying the Public Key to the Remote Server**
    
    - Run this command to copy your key into the server `ssh-copy-id user@example.com`.  
    - You will need to enter in a password to do this but from now on no more typing passwords!
    - If the previous didn't work you can try this method:
	    -  You can display and copy your public key with `cat ~/.ssh/id_rsa.pub`.
	    - Add this key to the `~/.ssh/authorized_keys` file on the remote server.

## 4. Some tips and tricks

You are using a desktop with 2 GPUs, by default when you run code, you will only use one so make you properly run code that uses the full capabilities of the computer!
Here's some basic ChatGPT generated guides on how to do this:
### TensorFlow

1.  **Install TensorFlow-GPU**: Ensure you have the GPU version of TensorFlow installed. This version is specifically built to utilize NVIDIA GPUs.
	```bash
	pip install tensorflow-gpu
	```
    
2.  **Check GPU Availability**: You can check if TensorFlow detects your GPUs using the following code:
    
    ```python
    import tensorflow as tf
    print("Num GPUs Available: ", len(tf.config.list_physical_devices('GPU')))
    ``` 
    
3.  **Enable GPU Growth**: This step is optional but recommended. It prevents TensorFlow from allocating the entire GPU memory at once.
    ```python
    gpus = tf.config.list_physical_devices('GPU')
    for gpu in gpus:
        tf.config.experimental.set_memory_growth(gpu, True)
    ``` 
    
4.  **Distribute Strategy**: Use TensorFlow's `tf.distribute.Strategy` API for distributing computations across multiple GPUs. The most common strategy is `MirroredStrategy`.
    

    
    ```python
    strategy = tf.distribute.MirroredStrategy()
    ``` 
    
5.  **Model Building and Training**: Encapsulate your model building and training code within the scope of the strategy:
  
    ```python
    with strategy.scope():
        # Build your model here
        model = ...
        model.compile(...)
    
        # Train your model here
        model.fit(...)
    ```

### PyTorch

1.  **Install PyTorch with CUDA**: Make sure you have the PyTorch version with CUDA support installed.
    
2.  **Check GPU Availability**: In PyTorch, you can check for available GPUs using:
    
    ```python
    import torch
    print(torch.cuda.device_count())
    ``` 
    
3.  **Data Parallelism**: Use `torch.nn.DataParallel` to parallelize your model over multiple GPUs. This wraps your model and automatically splits data across GPUs.
    
    ```python
    model = YourModel()
    if torch.cuda.device_count() > 1:
        print("Let's use", torch.cuda.device_count(), "GPUs!")
        model = torch.nn.DataParallel(model)
    model.to(device)
    ``` 
    
4.  **Adjust Your Training Loop**: Ensure your training loop correctly processes data on multiple GPUs. This usually involves ensuring data and model are on the same device.

    ```python
    for data in dataset:
        inputs, labels = data[0].to(device), data[1].to(device)
        # Rest of your training loop
    ```
<!--stackedit_data:
eyJwcm9wZXJ0aWVzIjoidGl0bGU6IEFwcGxlLVN1cGVyQ29tcH
V0ZXItd2lraVxuIiwiaGlzdG9yeSI6WzE5ODQ1MjE4MjYsMjQx
ODA4NzUyLDIwNDQ5MjExMTJdfQ==
-->
