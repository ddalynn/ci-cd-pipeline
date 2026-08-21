
CI/CD

Workflows 

00 - List Available DEV/RC/PROD Builds (Manual)
01 - CI (Auto)
02 - Build images (Auto)
02b - Build Poetry Base Image (Auto)
03 - Prepare Release (Manual)
04 - Release images to PROD (Manual)

*each workflow support manual trigger run 
the (Auto) and (Manual) are cite for the case of use of normal relase cycle 


Determine version from PR label.

Priority order:
major > minor > patch

If no active release cycle:
    Start new cycle with calculated version.

If active release cycle exists:
    Compare active cycle version vs incoming version.

    If incoming version has higher priority:
        Upgrade active cycle version.

    Else:
        Keep active cycle version.




Release cycle detection 

main = 3.3.2
integration = 3.3.2

Meaning: No active release cycle

CI detects: main v. == integration v . = starts a new cycle  > apply bump version > integration = 3.3.3

secound pr cycle already start 

main = 3.3.2
integration = 3.3.3
Meaning: Active release cycle started 
CI detects: main v. != integration v . = no bump version



## NORMAL RELEASE

Start

work1, work2, work3  > pr to intg with label by default: patch > auto trigger 01 CI - only check code | > 

INTG BRANCH ------------------

pr merge to INTG branch > auto trigger 01 CI - *detect release cycle - bump version - update version files > 01 done - auto trigger > 02 CD-build-image - build img - tag img dev, dev+sha, dev+ sha+date [ push to harbour DEV repo ] & 02b CD-Build-Poetry-Base-Image < pararelle run > 02,02b done - auto trigger  > 03 CD-Prepare Release - retag Images as RC eg. tag 3.3.3.rcN [ push to harbour UAT repo ] - auto PR to main  - manaul in put pr description for change log generation



pr merge to main branch >  | > 

MAIN BRANCH ----------------- 

auto trigger 01 CI - create tag, release - update change log  

 03 done - manual  trigger > 04 CD - Release images to PROD - Retag img to release version eg. 3.3.3,  3.3.3-sha, 3.3.3-sha-date [ push to harbour UAT/PROD repo ] 


------------- finish cycle release 




## HOT FIX/QUICK RELEASE

Start

HOTFIX/BRANCH --------- fix code done > 

pr to main branch >  auto trigger 01 CI - only check code | > 

 pr merge to Main branch | > 
 
 MAIN BRANCH -----------------  > auto trigger 01 CI - bump version for hot fix (patch )- update version files - create tag, release  > 01 done - manaul trigger > 02 CD-build-image - input tag, build img from that Hotfix tag >  02 done - manaul trigger > 04 CD - Release images to PROD - Retag img to release version eg. 3.3.3,  3.3.3-sha, 3.3.3-sha-date [ push to harbour UAT/PROD repo ]

------------- finish Hotfix release




Version priority: major > minor > patch 

 for example 
- current cycle = patch 3.3.4
- incoming PR = minor 3.4.0

keep -> minor
3.3.4 -> 3.4.0


## Scenario: Hotfix During Active Patch Release

Current state:

main        = 3.3.3
integration = 3.3.4

= Release cycle 3.3.4 is already active.
 
A production hotfix is needed, created and released.
then hotfix back merge to > main at 3.3.4 

> Result:

main        = 3.3.4
integration = 3.3.4

To avoid image tag conflicts, 
the active release cycle is automatically moved to the next patch version: integration = 3.3.5


