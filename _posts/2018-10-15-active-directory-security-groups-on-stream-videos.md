---
layout: post
title: Active Directory Security Groups on Stream Videos
tags: [office-365]
---

Using Microsoft Stream, I had a scenario where I needed a secure video that was limited to various departments in a company. These departments range from a handful of people to 100+ individuals. Fortunately, there were maintained Active Directory security groups which are replicated via Azure Active Directory Connect so the groups are available in Microsoft Stream. Unfortunately there were ~120 AD groups that were applicable to a handful of videos.

Since you cannot nest security groups within Office 365 groups, I had a couple of options.

1. Change the Office 365 Group to a Dynamic group (from Assigned) via the Azure Portal. I would have to build the rules out and rely on accurate information in Active Directory user objects.
2. Add the synchronized AD security groups to the videos directly.

While option 1 would have been ideal, it wasn't feasible in this particular environment. Option 2 will work, but it is painful due to the Stream UI as you can only add one group at a time into the permissions dialog box. So, in comes another option! I created another Active Directory security group and nested all of the other Active Directory security groups within it. Once that replicated via AAD Connect, I was able to add the new parent group to the Stream video and it worked!

Finally, to do this in Stream, you need to edit a video in Admin Mode. To do this, as an owner of the video/channel, go into the video itself, click the ellipsis under the video, and click Edit. There is a center panel for Permissions on the specific video. Change the drop down from My Groups to People. You can then find the security group and add it. Once you apply your changes, the video will be available for that group (or nested group :)) of users.