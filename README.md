# Ease SQL Server and SSIS job management
Here's how to avoid dealing with the SQL Agent Monitor and the SISS Package Catalog tree with all the mouse clicks and faff they require.

The SQL Server jobs that I have to look after are frequently troublesome and I find interacting with the SQL Agent Monitor and the Integration Services Catalogs very frustrating and tedious with their clunky structure and click-click-click required multiple times to see anything, plus in Agent Monitor you can only view one job's history at a time _and then_ your column settings will be lost each time you look at another job.  Much easier to see them from code in a query window.

The Queries below are what I use to list failed jobs and then dig into their failed steps:

```SQL
select * from Monitor.jobs.[fv_jobsForAttention]() order by 1, case when stepName like '%wait%' then 2 else 1 end, 2;
select * from Monitor.jobs.fv_History('Job name here!');
select * from [SSISDB].[catalog].[event_messages] where message_type in (120,130) and package_name = 'Package_name_here.dtsx' order by 2 desc, 1 asc;
```

(You need [fv_jobsForAttention](https://github.com/iywsdrdiy/fv_jobsForAttention) and [fv_History](https://github.com/iywsdrdiy/fv_History))

Then these can be built on further to ease up other tasks like restarting a failed job:
```SQL
select *, 'exec msdb.dbo.sp_start_job @job_name='''+jobName+''', @step_name='''+stepName+''';' restartStep from Monitor.jobs.fv_jobsForAttention() order by 1, case when stepName like '%wait%' then 2 else 1 end, 2;
```

Here you can see every package failure message, and this is usually good enough to work out what went wrong:
```SQL
select * from [SSISDB].[catalog].[event_messages] where message_type in (120,130) and operation_id in (select max(execution_id) execution_id from [SSISDB].[catalog].[executions] group by package_name) order by 2 desc, 1 desc
```
