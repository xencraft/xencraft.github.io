---
title: "Optimising Windows Laptop Power and Sleep Settings with Intune Remediations: Balancing Control and User Flexibility"
date: 2023-11-23 09:00:00 +0100
categories: [Intune, Remediations]
tags: [power options,intune,remediations,powershell]
img_path: /assets/windows-laptop-power-options-intune-remediation/
image: power-options-plan-settings.png
---

## Summary

In this article I will show you how you can set some Default Power and Sleep settings on Windows Laptops via Intune Remediations while still allowing users to change these settings based on their preference.

## Introduction

### The Importance of Power and Sleep Settings

Before we dive into the technical details, it's essential to understand why managing power and sleep settings is critical:

#### Energy Efficiency

Optimising power settings helps conserve energy, reducing utility costs and lowering your organisation's carbon footprint. It contributes to your corporate social responsibility efforts.

#### Device Health

Properly configured power and sleep settings can extend the lifespan of devices by preventing unnecessary wear and tear.

#### Productivity

Balancing power and sleep settings is crucial to ensure that devices are ready when users need them. Overly aggressive power settings can lead to frustrating delays when users return to their devices.

### The issue with Configuration Profiles

You might be thinking that we can simply use Configuration Profiles for this and it is pretty straight-forward; the main issue with setting power and sleep settings via configuration profiles is that they prevent users from making any modifications to these settings on their devices. This limitation can hinder our users ability to make a customisation that suits their workflow.

Our goal with these settings is to establish a default configuration for laptops while still allowing our user's the ability to change them if required.

We are only targeting laptops in this remediation as desktops are generally used differently and are often shared devices, however feel free to adapt or duplicate as needed.

## Methodology

To achieve this, we will leverage Intune Remediations (formerly known as Proactive Remediations). This approach involves executing a PowerShell script to adjust the Power and Sleep settings on the device for all the power plans on the device.

## Requirements

- Devices must be Azure AD joined or hybrid Azure AD joined
- Devices must be running Enterprise, Professional, or Education editions of Windows 10 or later
- Targeted users or devices must have an applicable license assigned. ([see here for details](https://learn.microsoft.com/en-us/mem/intune/fundamentals/remediations#licensing))

## Solution

#### Step 1: Download & Customise Scripts

1. Download the Detection and Remediation Scripts from [my Github](https://github.com/Xengrath/Scripts/tree/2c9f8a097c2307bb8ba250d42826e41bd8ed7d7e/Intune%20Proactive%20Remediations/Power%20Options%20Laptop)
2. Edit the PowerShell script and customise the values being configured for each option as needed

> You can learn more about using Powercfg.exe to control power schemes [here](https://learn.microsoft.com/en-us/windows-hardware/customize/power-settings/configure-power-settings#use-powercfgexe-to-control-power-schemes)
{: .prompt-info }


#### Step 2: Create a Remediation Script in Microsoft Intune

1. In the Intune admin center, navigate to: **Devices** > **Remediations**
2. Click **+ Create script package** button to create a script package
    
    ![intune-admin-center-create-script-package.png](intune-admin-center-create-script-package.png)
    
3. On the **Basics** tab, give the Remediation a **Name** and optionally, a **Description**. The **Publisher** field can be edited, but defaults to your name. **Version** can't be edited.
    
    ![create-remediation-basics-tab.png](create-remediation-basics-tab.png)
    
4. On the **Settings** tab, upload both the **Detection script file** and the **Remediation script file** you downloaded in the previous step
    
    ![create-remediation-settings-tab.png](create-remediation-settings-tab.png)
    
5. On the Scope tags tab, set any scope tags you use in your environment
6. On the assignments tab, target a device group and change the schedule to execute Once
    
    ![create-remediation-schedule-frequency.png](create-remediation-schedule-frequency.png)
    
7. Click Create once your are ready to deploy. Keep in mind that you should test of smaller groups of devices before doing a broad deployment!

#### Step 3: Monitor Remediation

Monitor the status of the remediation script in the Intune portal. This allows you to track which devices have been configured with the default power and sleep settings.

> Remember that you can add additional columns to the Device Status to see the output from the script: 
![remediation-results-select-columns.png](remediation-results-select-columns.png)
{: .prompt-tip }

- Devices that are laptops and have never had the remediation executed will report as follows:
    ![remediation-results-with-issues.png](remediation-results-with-issues.png)
    
- Devices that are not laptops will report as follows:
    ![remediation-results-without-issues.png](remediation-results-without-issues.png)
    
## Results

![power-options-system-settings.png](power-options-system-settings.png)
_Power Options > System Settings_

![power-options-plan-settings.png](power-options-plan-settings.png)
_Power Options > Plan Settings_

## Details

#### Detection Script Summary:

- The script determines if a device is a laptop by checking for the presence of a battery.
- It checks if power options have been applied previously by looking for the existence of a log file.
- If the device is a laptop and the log file exists, it outputs "Power Options have been applied" and exits with code 0.
- If the device is not a laptop, it outputs "Device is not a laptop" and exits with code 0.
- If the device is a laptop, but the log file doesn't exist, it outputs "Power Options have not been applied, Running Remediation..." and exits with code 1.

#### Remediation Script Summary:

- The script retrieves a list of all power plans and iterates through them to configure the specified power options.
- Retrieves a list of power plans on the device.
- Processes the output to extract power scheme GUIDs and names.
- Creates an array of custom objects containing GUID and Name for each power scheme.
- Attempts to update power settings for each power scheme.
- Configures settings for Turn Off Display After, Hard Disk Timeout, Sleep After, Hibernate After, Power Button Action, Sleep Button Action, and Close Lid Action.

## Conclusion

Leveraging Intune Remediations to configure default power and sleep settings on laptops is a powerful way to balance control and flexibility. It empowers IT administrators to promote energy efficiency and device health while allowing users to customise their settings as needed, ultimately contributing to a more efficient and sustainable IT environment. 

This approach not only helps you save energy and reduce costs but also enhances productivity and prolongs the life of your devices.