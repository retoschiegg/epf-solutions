```python
cron_trigger = entities.CronTrigger(expression="0 14 * * *")  # 15:00 CET
schedule = entities.JobSchedule(
    name="notebook-daily-epf-inference",
    trigger=cron_trigger,
    create_job=epf_inferencing_pipeline_job,
)
created_schedule_poller = ml_client.schedules.begin_create_or_update(schedule)
created_schedule = created_schedule_poller.result()
```
