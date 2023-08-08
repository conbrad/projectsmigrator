# Projects Migrator

Migrates one or more ZenHub workspaces to a Github Project

Usage:

```
python projectsmigrator.py https://github.com/orgs/myorg/myproj/1 -w="Workspace 1" --w="Workspace 2" -f="Estimate:Size" 
```

# Details

The project must be a ProjectV2 and must already exist and you should add the status and other field options you want manually first.

To see all the options look at https://raw.githubusercontent.com/pretagov/projectsmigrator/main/projectsmigrator.py
or run 

```
python projectsmigrator.py --help
```


## Columns/Status
The default setting is ```-f="Pipeline:Status"```
The columns/status value won't be added automatically if it doesn't exist but instead
the issues will be placed in the closest existing column that matches.

The position with in that column is set using the default ```-f="Position:Position"```.

You can drop pipelines via ```--exclude=Pipeline:MyIgnoredPipelines*```

You can change the closest option match behaviour to an exact match via ```-f="Pipeline:Status:Exact"```

## Epics

The default setting is ```-f="Epic:Text"```

This will recreate Epics as checklists in the form

``` markdown
# Dependencies

## Epic
- [ ] #23
- [ ] otherorg/otherepo#42
```

While Projects doesn't have native support for Epics, this does seem to be what githubs is leaning towards on the supported way to
related tickets togeather. When you view an issue that is a checklist elsewhere you will see this highlighted below the header and it
will include the headings "Dependencies Epic" so you can see why they are related.

One thing you lose is the ability to filter a board by Epic.

If you want you can instead convert Epics as a field on the sub issue.

```-f="Epic:MyEpicField"```. This will set the value to the name of the epic.

You can if you want do both at the same time. 

```-f="Epic:Text" -f="Epic:MyEpicField"```


## Blocking/Blocked

The default setting is ```-f="Blocking:Text"```

An issue that is blocked will have a dependencies section added to it that lists the blocking issues

e.g.
``` markdown
# Dependencies

## Blocked by
- [ ] #23
- [ ] otherorg/otherepo#42
```

Issues that are blocking will remain unchaged.

## Linked Pull Request

The default setting is ```-f="PR:Linked Pull Requests"```

Since zenhub has its own way to linking pull requests to issues this information is transfered by
modifying the PR to add text which then will use githubs automatic PR linking

e.g.
```
fixes otherorg/otherepo#42
```

it won't do this for PR's in repos outside the org your project is in

NOTE: Github linked PR's will automatically close the linked ticket once the PR is merged
to the main branch. This is different to zenhub. While you can't change this behavior, github
also doesn't move an item in your project automatically when it closes so this shouldn't change
your workflow too much.

## Workspace

You can specify which workspaces to merge using 
```
--workspace="Workspace 1" --workspace="Workspace 2"
```

or merge all workspaces excluding some
```
--exclude="Workspace:Workspace 1*"
```

They are merged in the priorty listed. ie if fields/pipeline data differs, the first workspace data will be used

If no workspaces are specified then all workspaces will be merged in most recently used order

You can optionally add the workspace name the issue came from into a new custom field, but ff not enabled this information is not transfered. This can be enabled with ```-f="Workspace:MyWorkspaceField"```


## Priority

The default setting is ```-f="Priority:Priority"```

By default Projects has a custom field called Priority with more options than zenhubs "High Priority" flag.
If this field exists we will migrate High Priority issues to the cloest word match.

## Estimate

The default setting is ```-f="Estimate:Size:Scale"```

The default Projects equivilent is "Size" which has a different scale to the default Zenhub story points.
This is mapped by rank. The 1st estimate option becomes the 1st size option. The last estimate option becomes the last
size option, and the rest are mapped proportionally.

NOTE: currently there doesn't seem to be an api to read the options for Estimate from zenhub so it assumes the default storypoints.
if the value doesn't match one of those then it will pick the closest story point.

## Sprints

You can enable transfer of sprint information with ```-f="Sprint:MySprintField"```.

If the field is a singleselect field it will pick the closest matching option.

## Authentication

You will need access to both the Zenhub graphql api and Github graphql api.
Once you have got auth tokens for both you can either 
- put them environment variables ```ZENHUB_TOKEN``` and ```GITHUB_TOKEN```
- use ```--zenhub-token=<token>``` and ```--github-token=<token>``` command line options

- TODO: Required token permissions
  
# TODO

- [ ] Dry run mode
- [ ] verbose or less verbose mode. esp to help see pipline mapping more clearly
- [ ] CSV import
- [ ] Migrate projects -> project
- [ ] filtering issues by src field value
- [ ] blacklist certain field values
- [ ] include labels as a field target e.g. put workspace name as new label
- [ ] handle sync data from closed issues
- [ ] Handle zenhub only epics and issues
- [ ] zenhub Milestones - depricated and not possible to get from the graphql
- [ ] zenhub Release Reports?
- [ ] Handle different estimate scales
- [ ] Fix inability to add more options to the workspace field or removing adding fields.
- [ ] set a global default value. e.g. normal priority

# Contributing

PR's or tickets welcome

# Credits

Sponsored by PretaGov UK/AU https://pretagov.com

Inspired by [this manual process for migrating zenhub to projects](https://medium.com/collaborne-engineering/migrate-from-zenhub-to-github-projects-948d69adc17d)
but we were motiviated by
- the need for longer transition so needed some way to sync so we needed something more automated
- the need to merge many workspaces into a single project
- the idea that epics represented by checklists rather than fields would work better
- the suprise that no one had written a tool for this already and that others might find it useful.

