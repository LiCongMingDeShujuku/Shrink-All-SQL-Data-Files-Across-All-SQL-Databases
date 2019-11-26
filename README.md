![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 压缩所有数据库中的所有文件
#### Shrink All SQL Files Across All SQL Databases
**发布-日期: 2014年05月02日 (评论)**

![#](images/##############?raw=true "#")

## Contents

- [中文](#中文)
- [English](#English)
- [SQL Logic](#Logic)
- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 


## 中文
这里有一些逻辑会缩小所有数据库中的所有日志文件。首先，它检查数据库是否处于完全恢复状态。这样我们就知道压缩文件操作是否有保证。如果检测到完全恢复，它将压缩日志文件。此外，该脚本仅针对当前“在线”的数据库执行此操作，因此它将跳过正在还原的任何脱机数据库。我通常只是把它放在一个作业中，然后每小时运行一次，来管理日志上的文件增长。


## English
Here’s some logic that will shrink all log files across all databases. First it checks to see if the database is in Full recovery. This way we know if the shrink file operation is even warranted. If Full Recovery is detected it will shrink the log files. Additionally the script will only do this against databases that are currently ‘online’, so it will skip any databases that are being restored, that are offline. I usually just put this in a Job and run it every hour to aggressively manage file growth on the logs.

---
## Logic
```SQL
use master;
set nocount on
declare @checkpoint_shrinkfile varchar(max)
set @checkpoint_shrinkfile = ''
select @checkpoint_shrinkfile = @checkpoint_shrinkfile +
'use [' + [master].sys.databases.name + ']; checkpoint; waitfor delay ''00:00:01'';' + char(10) + '
declare @recovery_type_' + cast([master].sys.databases.database_id as varchar) + ' varchar(50);' + char(10) + '
set @recovery_type_' + cast([master].sys.databases.database_id as varchar) + ' = ( select recovery_model_desc from master.sys.databases where database_id = ''' + cast([master].sys.databases.database_id as varchar) + ''' );' + char(10) + '
if @recovery_type_' + cast([master].sys.databases.database_id as varchar) + ' = ''FULL'' collate Latin1_General_CI_AS_KS_WS' + char(10) + ' begin ' + char(10) + '
dbcc shrinkfile (' + cast(sys.master_files.file_id as varchar) + ', 8)' + char(10) + 'print ''The shrinkfile operation has completed for Database [' + replace(db_name([master].sys.databases.database_id), '''', '') + ']''' + char(10) + '
end
else
print ''The shrinkfile operation is not required because the Recovery Model is SIMPLE on Database: [' + replace(db_name([master].sys.databases.database_id), '''', '') + ']'';' + char(10) + '' + char(10) + '' from
[master].sys.master_files join [master].sys.databases on
[master].sys.master_files.database_id = [master].sys.databases.database_id where
[master].sys.master_files.type_desc = 'log'
and [master].sys.databases.name not in ('master', 'model', 'msdb', 'tempdb') and [master].sys.databases.state_desc = 'online';
exec (@checkpoint_shrinkfile)


```

希望对你有所帮助。(Hope you find this helpful.)



[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")

