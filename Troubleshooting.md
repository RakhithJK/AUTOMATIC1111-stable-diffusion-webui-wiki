- ### **The program is tested to work on Python 3.10.6. Don't use other versions unless you are looking for trouble.**
- The program needs 16gb of regular RAM to run smoothly. If you have 8gb RAM, consider making an 8gb page file/swap file, or use the [--lowram](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Command-Line-Arguments-and-Settings) option (if you have more gpu vram than ram).
- The installer creates a python virtual environment, so none of the installed modules will affect existing system installations of python.
- To use the system's python rather than creating a virtual environment, use custom parameter replacing `set VENV_DIR=-`.
- To reinstall from scratch, delete directories: `venv`, `repositories`.
- When starting the program for the first time, the path to python interpreter is displayed. If this is not the python you installed, you can specify full path in the `webui-user` script; see [Command-Line-Arguments-and-Settings#environment-variables](Command-Line-Arguments-and-Settings#environment-variables).
- If the desired version of Python is not in PATH, modify the line `set PYTHON=python` in `webui-user.bat` with the full path to the python executable.
    - Example: `set PYTHON=B:\soft\Python310\python.exe`
- Installer requirements from `requirements_versions.txt`, which lists versions for modules specifically compatible with Python 3.10.6. If this doesn't work with other versions of Python, setting the custom parameter `set REQS_FILE=requirements.txt` may help.

# Run `webui-user.bat` but the window immediately closes
Some error has occurred but the window closed too fast we can't see the issue, we need to prevent `webui-user.bat` form closeing instantly.
1. Right click to edit `webui-user.bat`.
2. Add a command `pause` at the end of the file.
3. Save the modification and run `webui-user.bat` again.

`webui-user.bat` should now pause allowing you to see the issue.

<details><summary>The modified file should look similar to this. (Click to expand)</summary>
<p>

```bat
@echo off
set PYTHON=
set GIT=
set VENV_DIR=
set COMMANDLINE_ARGS=

call webui.bat
pause
```

</p>
</details> 

# Launching as root
Running as root is not recommended, but can be overriden by adding `-f` to your arguments.
```bash
./webui.sh -f
```

# Low VRAM Video-cards
When running on video cards with a low amount of VRAM (<=4GB), out of memory errors may arise.
Various optimizations may be enabled through command line arguments, sacrificing some/a lot of speed in favor of using less VRAM:
- Use `--opt-sdp-no-mem-attention` OR the optional dependency `--xformers` to cut the gpu memory usage down by half on many cards.
- If you have 4GB VRAM and want to make ~1.3x larger images, use `--medvram`.
- If you have 4GB VRAM but you get an out of memory error with `--medvram`, use `--lowvram --always-batch-cond-uncond` instead.
- If you have 4GB VRAM and want to make images larger than you can with `--medvram`, use  `--lowvram`.
- If you have 4GB VRAM and get an out of memory error when loading a full weight model, use `--disable-model-loading-ram-optimization` (added in v1.6.0)

# Torch is not able to use GPU
```
Torch is not able to use GPU; add --skip-torch-cuda-test to COMMANDLINE_ARGS variable to disable this check
```
This is one of the most frequently mentioned problems, but it's usually not a WebUI fault, there are many reasons for it.

- WebUI uses GPU by default, and to make sure that GPU is working is working correctly we perform a test to see if CUDA is available, CUDA is only available on NVIDIA GPUs, so if you don't have a NVIDIA GPU or if the card is too old you might see this message.
- If you encountered this message with a Navidad graphics card, then something is wrong, then something has gone wrong and is preventing whether you are from using your GPU properly.
- However if you're using different Hardware such as an AMD GPU or a Mac then this message is expected, you should add `--skip-torch-cuda-test` to COMMANDLINE_ARGS as to bypass this CUDA test and follow the installation guide for you are specific hardware.
 your specific hardware, and 
if you don't have any hardware acceleration then the only option for you is to run on to [run on CPU](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Command-Line-Arguments-and-Settings#running-on-cpu).
- Make sure you configure the WebUI correctly, refer to the corresponding installation tutorial in the [wiki](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki).
- If you encounter this issue after some component updates, try undoing the most recent actions.

If you are one of the above, you should delete the `venv` folder.

If you still can't solve the problem, you need to submit some additional information when reporting.
1. Open the console under `venv\Scripts`
2. Run `python -m torch.utils.collect_env`
3. Copy all the output of the console and post it

# Green or Black screen
Video cards
Certain GPU video cards don't support half precision: a green or black screen may appear instead of the generated pictures. Use `--upcast-sampling`. This should stack with `--xformers` if you are using.
If still not fixed, use command line arguments `--precision full --no-half` at a significant increase in VRAM usage, which may require `--medvram`.

# A Tensor with all NaNs was produced in the vae
This is the same problem as the one from above, to verify, Use `--disable-nan-check`. With this on, if one of the images fail the rest of the pictures are displayed. 

It is either a model cause - [resource](https://github.com/arenasys/stable-diffusion-webui-model-toolkit#clip)

Merge cause - [resource](https://github.com/Mikubill/sd-webui-controlnet/discussions/1214)

Or GPU related.
- NVIDIA 16XX and 10XX cards should be using --upcast-sampling and --xformers to run at equivalent speed. If issue is persisting, try running the vae in fp32 by adding `--no-half-vae` If this fails, you will have to fall back to running with `--no-half`, which would be the the slowest + using the most gpu memory.

- AMD cards that cannot run fp16 normally should be on `--upcast-sampling --opt-sub-quad-attention` / `--opt-split-attention-v1`. The fallback order should ideally be the same as the one above. Following that, if it continues to fail, AMD users may need to utilize some trick like "export HSA_OVERRIDE_GFX_VERSION=10.3.0" specific to their GPU. It would be ideal to do a thorough google search + all of github search to find the HSA_OVERRIDE_GFX_VERSION right for your specific GPU.

(These are word-of-mouth troubleshooting tips. Test with a fp32 4gb SD1 model)

# "CUDA error: no kernel image is available for execution on the device" after enabling xformers
Your installed xformers is incompatible with your GPU. If you use Python 3.10, have a Pascal or higher card and run on Windows, add `--reinstall-xformers --xformers` to your `COMMANDLINE_ARGS` to upgrade to a working version. Remove `--reinstall-xformers` after upgrading.

# NameError: name 'xformers' is not defined
If you use Windows, this means your Python is too old. Use 3.10

If Linux, you'll have to build xformers yourself or just avoid using xformers.

# `--share` non-functional after gradio 3.22 update

Windows defender/antiviruses sometimes blocks Gradio's ability to create a public URL.

1. Go to your antivirus
2. Check the protection history: \
![image](https://user-images.githubusercontent.com/98228077/229028161-4ad3c837-ae3f-45f7-9a0a-fa165d70d943.png)
3. Add it as an exclusion

Related issues:
<details>

https://github.com/gradio-app/gradio/issues/3230 \
https://github.com/gradio-app/gradio/issues/3677
</details>

# weird css loading

![image](https://user-images.githubusercontent.com/98228077/229085355-0fbd56d6-fe1c-4858-8701-6c5697b9a6d6.png)

This issue has been noted, 3 times. It is apparently something users in china may experience.
[#8537](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/8537)

<details><summary> Solution: </summary>

This problem is caused by errors in the CSS file type information in my computer registry, which leads to errors in CSS parsing and application.
Solution:

![image](https://user-images.githubusercontent.com/98228077/229086022-f27858a3-c9d9-470c-87cc-aa1974b7c5d0.png)


According to the above image to locate, and modify the last Content Type and PerceivedType.
Finally, reboot the machine, delete the browser cache, and force refresh the web page (shift+f5).
Thanks to https://www.bilibili.com/read/cv19519519
</details>

# fatal: not a git repository

When running a git command some users may encounter this issue
```
fatal: not a git repository (or any of the parent directories): .git
```

This happes to some user when ther download webui by using `Downloading ZIP` button on GitHub's webpage, this button download the code without the `.git` dir which contains the metadata of a git repo, 
as it is not a git repository git operations doesn't work.

## Fix: convert a non-git dir to a git repo

This can be fixed by convert a non-git dir to a git repo by running the following command in the `webui root directory`

### Warnings
> If you do not have a basic understanding of running commands into a terminal

> You MUST make sure your running the command in the webui's root directory<br>

> If you're applying this guide for some other project that is NOT [AUTOMATIC1111/stable-diffusion-webui](https://github.com/AUTOMATIC1111/stable-diffusion-webui) You MUST NOT use the command as is<br>
> You MUST MODIFY the command appropriately, especially the `root directory` `remote URL` and `branch name`<br>

> **failingly above may result in unintended data loss**


```
cd "<path-=of-webui-root>"
git init
git remote add origin https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
git fetch origin
git switch master --force
```

meaning of each line of command

0. change directory to the webui's root directory
   - this change the directory where the commands will be applied to
1. initialize the `current directory` as a `git repo`, (adding the .git dir and it's contents)
2. set the remote of the git repo as `https://github.com/AUTOMATIC1111/stable-diffusion-webui.git`
   - if you're following this guide for some other project you must modify the remote URL appropriately
3. fecth git metadata from the remote origin
4. switch the repo onto the `master` branch
   - `--force` is used so that it will automatically override any conflicting files
   - use different bench name other then `master` if you wish to switch to other branch, for example `dev` branch
