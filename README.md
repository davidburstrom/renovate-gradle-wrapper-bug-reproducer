# Reproduces https://github.com/renovatebot/renovate/discussions/26050

TLDR: Run `./gradlew :wrapper` instead of `./gradle wrapper`, here's the reproducer: https://github.com/davidburstrom/renovate-gradle-wrapper-bug-reproducer

When Renovate updates the Gradle wrappers, it runs using the `wrapper` tasks (https://github.com/renovatebot/renovate/blob/main/lib/modules/manager/gradle-wrapper/artifacts.ts#L149C15-L149C15).

For certain projects, such as https://github.com/davidburstrom/version-compatibility-gradle-plugin, the subprojects are configured to check that the current Gradle version is listed in a set of Gradle versions for compatibility testing. The Gradle configuration fails when Renovate runs, emitting a subtle `WARN: Error executing gradle wrapper update command. It can be not a critical one though.` in the logs, and thus it never updates the wrapper JAR and wrapper scripts.

Renovate really should be running `:wrapper` instead (note the absolute path to the task), which, if the project has turned on [configuration on demand](https://docs.gradle.org/current/userguide/multi_project_configuration_and_execution.html#sec:configuration_on_demand), will only configure the root project.

Obviously, when the PR actually runs in CI it will fail for the same reason, but by that time the relevant Gradle files have been updated at least.
A minor benefit of this change is that the renovate job runs slightly faster too. 
