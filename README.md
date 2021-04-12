# Minion::Task::Generator - a plugin for generating Minion tasks and jobs 

## Description

Minion::Task::Generator allows flexible creation of tasks with pluggable and configurable roles. 

## Synopsis

    use Minion::Task::Generator qw/task/;

    app->minion->add_task(some_task => task(sub {
                                       my $job = shift;
                                       sleep 5;
                                       $job->app->log->info('I am a task');
                                       sleep 5;
                                       return 'Done!';
                                   },
                                   {
                                    roles => {
                                              '+Alerter' => {
					                      alert_on => [qw/finish fail/ ],
							      url      => 'http://127.0.0.1:3000/status'
							    },
                                              '+Timeout' => { timeout => 6 }
                                             }
                                   }));



## Methods

### new

    my $task = Minion::Task::Generator->new(sub {}, { roles => {} })

Creates a new task that executes the subref passed as the first argument, and the options as the second.

The "roles" option defines which roles will be applied to the job on execution; if a role begins with "+", "Minion::Job::Role" gets prepended to the role name in place of the plus sign.

The hashref values of the roles option get passed to the role on job execution. 

### task

    my $task = task(sub {}, { roles => {} })

Shorthand for Minion::Task::Generator->new()

## Passing options to the job

Options can be passed

- when the task gets created
- when the job gets queued

To pass options to the role, they are defined as part of the roles hashref, as the value to the key that defines the role.

In this case:

    app->minion->add_task(some_task => task(sub { return 1 },
                                   {
                                    roles => {
                                              '+Timeout' => { timeout => 6 }
                                             }
                                   }));

job will be run with the Minion::Job::Role::Timeout role. The role will be passed a timeout option of 6 seconds.

Options can also be overridden at the job level, by passing them after an argument named '-opts' like this:

    app->minion->enqueue('some_task',  [
                                       'argument 1',
                                       'argument 2',
                                       -opts => { timeout => 12 }
                                       ],
                                       { priority => 1 });

In this case the timeout option at job level will override the general one set when the task was created.
