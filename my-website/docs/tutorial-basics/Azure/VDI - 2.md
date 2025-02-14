---
sidebar_position: 2
---

# AZURE Virtual Desktop (AVD) round 2

## Workspace

    - Alright! What we need now is a Workspace. (A virtual desktop workspace is a logical grouping of application groups, allowing users to access the desktops and applications published)
        ![Workspace](workspace.png)

    - We add the application group :
        ![appgroup](appgroup.png)

    - No advance nor tags

    - And Voilà, Workspace created.

## Acces to users and admins

    - Now, need to give acces to the right people. So going back to the ressource group, let's go to acces control (IAM) and click add role assignment :
        ![roleassign](roleassign.png)

    - Then assign the right group. In this case, I created a Test group :
        ![Testgroup](testgroup.png)

    - The assignment type is Active and permanent because I want this group to access the desktop at anytime :
        ![assignmentUsers](AssignmentUser.png)

    - Then I'll assign the admin group :
        ![assignmentAdmin](assignemntadmin.png)

        - Note : **Since this setup has PIM activated, I used the eligible option.** (But it doesn't seem to work)
        