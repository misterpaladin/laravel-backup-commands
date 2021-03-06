# lumen-backup-commands
Backup commands for backing up the database and project files (complete project dir).<br>
Works on \*NIX yet. Windows needs archiver support integrations.<br>
Database backup supports only PostgreSQL and MySQL. Wanna more - make issue with suggestions!<br>
Commands supports Verbosity Level 64 (-v flag) to show output (default is quiet)

## Installation

### composer

```bash
composer require h-zone/lumen-backup-commands:~0.1-dev
```

or

`composer.json`
```json
"h-zone/lumen-backup-commands": "~0.1-dev"
```

### Lumen
See https://github.com/h-zone/lumen-backup-commands

### Laravel 5.2+
`config/app.php`
```php
'providers' => [
    //....
    Hzone\BackupCommands\Providers\BackupCommandsServiceProvider::class,
    //....
],
```

### Publish config
```sh
php artisan vendor:publish --provider="Hzone\BackupCommands\Providers\BackupCommandsServiceProvider" --tag="config"
```

### Usage
```sh
php artisan backup-commands:database
php artisan backup-commands:database --database=all
php artisan backup-commands:database --database=dbalias
php artisan backup-commands:files
```
Or with output (verbosity check)
```sh
php artisan backup-commands:database -v
php artisan backup-commands:database -v --database=all
php artisan backup-commands:database -v --database=dbalias
php artisan backup-commands:files -v
```

### Scheduling
According to the https://laravel.com/docs/5.2/scheduling

app/Console/Kernel.php
```php
<?php namespace App\Console;
use Illuminate\Console\Scheduling\Schedule;
use Laravel\Lumen\Console\Kernel as ConsoleKernel;
class Kernel extends ConsoleKernel
{
	protected $commands = [
		Hzone\BackupCommands\Console\Commands\BackupDatabaseCommand::class,
		Hzone\BackupCommands\Console\Commands\BackupFilesCommand::class,
	];
	protected function schedule(Schedule $schedule)
	{
		$schedule->command('backup-commands:database -v')
			->withoutOverlapping()
			->appendOutputTo(storage_path().'/logs/backup-database-commands_'.date('Y-M-D_H-i-s').'.log')
			->dailyAt('04:00');
		$schedule->command('backup-commands:files -v')
			->withoutOverlapping()
			->appendOutputTo(storage_path().'/logs/backup-files-commands_'.date('Y-M-D_H-i-s').'.log')
			->weekly();

		/**
		 * IF YOU NEED TO BACKUP A SPECIFIC DATABASE IN A SPECIFIC TIME,
		 * USE the --database=dbalias DESIGN
		 * 
		 * $schedule->command('backup-commands:database -v --database=dbalias1')
		 * 	->withoutOverlapping()
		 * 	->appendOutputTo(storage_path().'/logs/backup-database-commands_dbalias1_'.date('Y-M-D_H-i-s').'.log')
		 * 	->dailyAt('04:00');
		 *
		 * $schedule->command('backup-commands:database -v --database=dbalias2')
		 * 	->withoutOverlapping()
		 * 	->appendOutputTo(storage_path().'/logs/backup-database-commands_dbalias2_'.date('Y-M-D_H-i-s').'.log')
		 * 	->monthly();
		 * 
		 */
	}
}

```

### BACKUP FILES BEFORE COMPOSER UPDATE
composer.json
```json
"scripts": {
    "pre-update-cmd": [
        "php artisan backup-commands:databases -v",
        "php artisan backup-commands:files -v"
    ]
}
```

#### Optional
Optional cleanup before `composer update`
`composer.json`
```json
"scripts": {
    "pre-update-cmd": [
        "php artisan cleanup-commands:view-cache",
        "php artisan cleanup-commands:logs"
    ]
}
```
See https://github.com/h-zone/lumen-cleanup-commands

