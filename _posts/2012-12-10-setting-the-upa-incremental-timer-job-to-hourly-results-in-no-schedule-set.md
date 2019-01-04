---
layout: post
title: Setting the UPA Incremental Timer Job to Hourly Results in “No Schedule Set”
tags: [sp2010]
---

If you set the User Profile Services’ Incremental Timer Job to run more frequently than “Daily”, the User Profile Service Application will indicate “No Schedule Set”:

![image3](/assets/images/2012/12/image3.png)

This is because the code that reviews the timer job doesn’t take into account the Incremental job running more often than daily.

```csharp
public static string GetScheduleString(UserProfileApplicationJob job)
{
    string str = Microsoft.SharePoint.Portal.WebControls.StringResourceManager.GetString(LocStringId.ProfileAdminMain_ProfImp_Schedule_LinkText_Text); //"No Schedule Set"
    try
    {
        using (ServerApplication.BeginSecurityContext())
        {
            if ((job == null) || job.IsDisabled)
            {
                return Microsoft.SharePoint.Portal.WebControls.StringResourceManager.GetString(LocStringId.ProfileAdminMain_ProfImp_Schedule_Disable_Text);
            }
            if (!(job.Schedule is SPOneTimeSchedule))
            {
                if (job.Schedule is SPMonthlySchedule)
                {
                    SPMonthlySchedule schedule = job.Schedule as SPMonthlySchedule;
                    DateTime dateTimeForSchedule = ScheduledJobHelper.GetDateTimeForSchedule(schedule.BeginDay, schedule.BeginHour);
                    return string.Format(CultureInfo.CurrentCulture, Microsoft.SharePoint.Portal.WebControls.StringResourceManager.GetString(LocStringId.Schedule_Monthly_Text), new object[] { dateTimeForSchedule.Day.ToString(CultureInfo.InvariantCulture), GetHourFormattedString(dateTimeForSchedule.Hour) });
                }
                if (job.Schedule is SPWeeklySchedule)
                {
                    SPWeeklySchedule schedule2 = job.Schedule as SPWeeklySchedule;
                    DateTime time2 = ScheduledJobHelper.GetDateTimeForSchedule(schedule2.BeginDayOfWeek, schedule2.BeginHour);
                    return string.Format(CultureInfo.CurrentCulture, Microsoft.SharePoint.Portal.WebControls.StringResourceManager.GetString(LocStringId.Schedule_Weekly_Text), new object[] { GetDayOfWeek(time2.DayOfWeek), GetHourFormattedString(time2.Hour) });
                }
                if (job.Schedule is SPDailySchedule)
                {
                    SPDailySchedule schedule3 = job.Schedule as SPDailySchedule;
                    DateTime time3 = ScheduledJobHelper.GetDateTimeForSchedule(schedule3.BeginHour);
                    str = string.Format(CultureInfo.CurrentCulture, Microsoft.SharePoint.Portal.WebControls.StringResourceManager.GetString(LocStringId.Schedule_Daily_Text), new object[] { GetHourFormattedString(time3.Hour) });
                }
            }
            return str;
```

While I don’t believe this will impact the job running more than once a day, it will impact the display of the schedule on the User Profile Service Application page.  Depending on the size of the organization, other jobs running, and performance issues with the Synchronization database, it is probably unwise to run the User Profile Incremental Synchronization timer job more than once per day.