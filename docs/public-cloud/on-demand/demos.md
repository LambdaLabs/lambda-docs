# Demos

The Demos feature allows you to easily share your [Gradio](https://gradio.app/) machine learning app (demo) both publicly and privately.

To get started using the Demos feature, you need to:

1. [Add a demo to your Lambda GPU Cloud account](demos.md#add-a-demo-to-your-lambda-gpu-cloud-account).
2. Host your demo on a new instance.

!!! note

    It currently isn’t possible to host a demo on an existing instance.

!!! note

    The new instance hosting your demo can be used like any other Lambda GPU Cloud on-demand instance. For example, you can SSH into the instance and [open Jupyter Notebook](https://old.docs.lambdalabs.com/cloud/open-jupyter-notebook/) on the instance.

    As with other Lambda GPU Cloud on-demand instances, [you’re billed for all of the time the instance for your demo is running](https://old.docs.lambdalabs.com/cloud/on-demand-instance-invoicing/).

!!! note

    The Demos feature can be hosted on multi-GPU instance types. However, Demos uses only one of the GPUs.

    Also, demos currently can’t be hosted on H100 instances.

## Add a demo to your Lambda GPU Cloud account <a href="#add-a-demo-to-your-lambda-gpu-cloud-account" id="add-a-demo-to-your-lambda-gpu-cloud-account"></a>

1.  In the left sidebar of the [dashboard](https://cloud.lambdalabs.com/instances), click **Demos**. Then, click the **Add demo** button at the top-right of the dashboard.

    The **Add a demo** dialog will appear.

2.  Under **Demo Source URL**, enter the URL of the Git repository containing your demo’s source code.

    **Note**

    The Demos feature looks in your Git repository for a file named `README.md`. If the file doesn’t exist, or if the file doesn’t contain the required properties, you’ll receive a **Demo misconfigured** error.


    The `README.md` _must_ have at the top a YAML block containing the following:

    ```yaml
    ---
    sdk: gradio
    sdk_version: GRADIO-VERSION
    app_file: PATH-TO-APP-FILE
    ---
    ```

    Replace **GRADIO-VERSION** with the version of Gradio your demo is built with, for example, `3.24.1`.

    Replace **PATH-TO-APP-FILE** with the path to your Gradio application file (the file containing the Gradio [interface code](https://gradio.app/docs/#interface)), relative to the root of your Git repository. For example, if your Gradio application file is named `app.py` and is located in the root directory of your Git repository, replace **PATH-TO-APP-FILE** with `app.py`.

    Properties other than `sdk`, `sdk_version`, and `app_file` are ignored by the Demos feature.

    **Tip**

    If you don’t yet have your own demo, you can try the Demos feature using the [demos created by Lambda’s Machine Learning team](https://huggingface.co/lambdalabs). Demos created by Lambda’s Machine Learning team include:

    * [Stable Diffusion Image Variations](https://huggingface.co/spaces/lambdalabs/stable-diffusion-image-variations)
    * [Image Mixer](https://huggingface.co/spaces/lambdalabs/image-mixer-demo)
    * [Avatar text to image](https://huggingface.co/spaces/lambdalabs/text-to-avatar)
3. Under **Visibility**, choose:
   * **Public** if you want to list your demo in the [library of public models shared by the Lambda community](https://cloud.lambdalabs.com/demos).
   * **Unlisted** if you want your demo accessible only by those who know your demo’s URL.
4. Under **Name**, give your demo a name. If you choose to make your demo public, the name of your demo will appear in the Lambda library of public models. The name of your demo will also appear in your demo’s URL.
5.  (Optional) Under **Description**, enter a description for your demo.

    The description shows under the name of your demo in your library of demos. If your demo is public, the description also shows under the name of your demo in the Lambda library of public models.

    **Note**

    You can’t change the name or description of your demo once you add it. However, you can delete your demo then add it again.
6.  Click **Add demo**, then follow the prompts to launch a new instance to host your demo.


!!! tip

    To host a demo that’s already added to your account, in the [Demos dashboard](https://cloud.lambdalabs.com/edit-demos), find the row containing the demo you want to host, then click **Host**.

[Your new instance will take several minutes to launch](getting-started.md#how-long-does-it-take-for-instances-to-launch) and for your demo to become accessible.

!!! note

    The link to your demo might temporarily appear in the Instances dashboard, then disappear. This is expected behavior and doesn’t mean your instance or demo is broken.

    The models used by demos are often several gigabytes in size, and can take 5 to 15 minutes to download and load.

1. Once your instance is launched and your demo is accessible, a link with your demo’s name will appear under the **Demo** column. Click the link to access your demo.

## Troubleshooting demos <a href="#troubleshooting-demos" id="troubleshooting-demos"></a>

If you experience trouble accessing your demo, the Demos logs can be helpful for troubleshooting.

To view the Demos log files, SSH into your instance or open a terminal in [Jupyter Notebook](getting-started.md#how-do-i-open-jupyter-notebook-on-my-instance), then run:

```bash
sudo bash -c 'for f in /root/virt-sysprep-firstboot.log ~demo/bootstrap.log; do printf "### BEGIN $f\n\n"; cat $f; printf "\n### END $f\n\n"; done > demos_debug_logs.txt; printf "### BEGIN journalctl -u lambda-demos.service\n\n$(journalctl -u lambda-demos.service)\n\n### END journalctl -u lambda-demos.service" >> demos_debug_logs.txt'
```

This command will produce a file named `demos_debug_logs.txt` containing the logs for the Demos feature. You can review the logs from within your instance by running `less demos_debug_logs.txt`. Alternatively, you can download the file locally to review or share.

!!! note

    The [Lambda Support](https://lambdalabs.com/support) team provides only basic support for the Demos feature. However, assistance might be available in the [community forum](https://deeptalk.lambdalabs.com/).

    If you’re experiencing problems using the Demos feature, running the above command and [providing the `demos_debug_logs.txt` file to the Support team](https://support.lambdalabs.com/hc/en-us/requests/new) can help with future improvements to the Demos feature.

Here are some examples of how problems present in logs:

### Misconfigured README.md file <a href="#misconfigured-readmemd-file" id="misconfigured-readmemd-file"></a>

```
### BEGIN /home/demo/bootstrap.log

Cloning into '/home/demo/source'...
Traceback (most recent call last):
  File "<stdin>", line 17, in <module>
  File "<stdin>", line 15, in load
  File "pydantic/main.py", line 526, in pydantic.main.BaseModel.parse_obj
  File "pydantic/main.py", line 341, in pydantic.main.BaseModel.__init__
pydantic.error_wrappers.ValidationError: 3 validation errors for Metadata
sdk
  field required (type=value_error.missing)
sdk_version
  field required (type=value_error.missing)
app_file
  field required (type=value_error.missing)
Created symlink /etc/systemd/system/multi-user.target.wants/lambda-demos-error-server.service → /etc/systemd/system/lambda-demos-error-server.service.
Bootstrap failed: misconfigured

### END /home/demo/bootstrap.log
```

### Not a Gradio app <a href="#not-a-gradio-app" id="not-a-gradio-app"></a>

```
### BEGIN /home/demo/bootstrap.log

Cloning into '/home/demo/source'...
Traceback (most recent call last):
  File "<stdin>", line 17, in <module>
  File "<stdin>", line 15, in load
  File "pydantic/main.py", line 526, in pydantic.main.BaseModel.parse_obj
  File "pydantic/main.py", line 341, in pydantic.main.BaseModel.__init__
pydantic.error_wrappers.ValidationError: 2 validation errors for Metadata
sdk
  unexpected value; permitted: 'gradio' (type=value_error.const; given=docker; permitted=('gradio',))
sdk_version
  field required (type=value_error.missing)
Created symlink /etc/systemd/system/multi-user.target.wants/lambda-demos-error-server.service → /etc/systemd/system/lambda-demos-error-server.service.
Bootstrap failed: misconfigured

### END /home/demo/bootstrap.log
```
