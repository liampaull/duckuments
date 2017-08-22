# Configuration

This chapter explains what are the assumptions about the configuration.

While the "Setup" parts are "imperative" (do this, do that); this is the
"declarative" part, which explains what are the properties of a correct
configuration (but it does not explain how to get there).

he tool `what-the-duck` ([](#what-the-duck)) checks some of these conditions.
If you make a change from the existing conditions, make sure that it gets
implemented in `what-the-duck` by filing an issue.

## Environment variables {#variabes}

You need to have set up the variables in [](#tab:environment-variables).

<col3 figure-id="tab:environment-variables" class='labels-row1'>
    <figcaption>Environment variables used by the software</figcaption>

    <s>variable</s>
    <s>reasonable value</s>
    <s>contains</s>

    <s><code>DUCKIETOWN_ROOT</code></s>
    <s><code>~/duckietown</code></s>
    <s><code>Software</code> repository</s>

    <s><code>DUCKIEFLEET_ROOT</code></s>
    <s><code>~/duckiefleet</code></s>
    <s>A repository that contains <code>scuderia.yaml</code> and other
    team-specific configuration. </s>

    <s><code>DUCKIETOWN_DATA</code></s>
    <s><code>~/duckietown-data</code></s>
    <s>Contains data for unit tests (Dropbox folder)</s>

    <s><code>DUCKIETOWN_CONFIG_SEQUENCE</code></s>
    <s><code>defaults:baseline:vehicle:user</code></s>
    <s>The <a href="#easy_node">configuration sequence for EasyNode</a></s>
</col3>

<style>
#tab\:environment-variables {
    font-size: 80%;
}
#tab\:environment-variables td {
    text-align: left;
}
#tab\:environment-variables td:nth-child(3) {
    display: block;
    width: 20em;
}
</style>

### Duckietown root directory `DUCKIETOWN_ROOT`

TODO: to write

### Duckiefleet directory `DUCKIEFLEET_ROOT`

For Fall 2017, this is the the repository [`duckiefleet-fall2017`][duckiefleet-repo].

For self-guided learners, this is an arbitrary repository to create.

[duckiefleet-repo]: https://github.com/duckietown/duckiefleet-fall2017

## The scuderia file {#scuderia}

<!-- do not change the ID "scuderia", it's linked in the code -->

In the <code>&#36;{DUCKIEFLEET_ROOT}</code> directory,
there needs to exist a file called:

    ${DUCKIEFLEET_ROOT}/scuderia.yaml

The file must contain YAML entries of the type:

    ![robot-name]:
       username: ![username]
       owner_duckietown_id: ![owner duckietown ID]

A minimal example is in [](#code:scuderia-minimal).

<div figure-id="code:scuderia-minimal" markdown="1">
<figcaption>Minimal scuderia file</figcaption>
``` .yaml
emma:
  username: andrea
  owner_duckietown_id: censi
```
</div>

Explanations of the fields:

* `![robot name]`: the name of the robot, also equal to the host name.
* `![username]`: the name of the Linux user on the robot, from which to run programs.
* `![owner_duckietown_id]`: the owner's globally-unique Duckietown ID.

## The `machines` file {#machines}

<!-- do not change the ID "machines"; it's linked in the code -->

The `machines` file is created using:

    $ rosrun duckietown create-machines-file

## People database {#people-file}

Assigned: Andrea


TODO: Describe the people database; this is the evolution of the yaml files


### The globally-unique Duckietown ID

This is a globally-unique ID for people in the Duckietown project.

It is equal to the Slack username.


## Modes of operation

There are 3 modes of operation:

1. `MODE-normal`: Everything runs on the robot.
2. `MODE-offload`: Drivers run on the robot, but heavy computation runs on the laptop.
3. `MODE-bag`: The data is provided from a bag file, and computation runs on the laptop.

<!-- 4. `MODE-unittest`: This is the case where many unit tests are run in parallel, on the cloud.  -->

<col4 class='labels-row1' id='operation-modes' figure-id="tab:operation-modes" figure-caption="Operation modes">

    <s>mode name</s>
    <s>who is the ROS master</s>
    <s>where data comes from</s>
    <s>where heavy computation happen</s>

    <code>MODE-normal</code>
    <code>duckiebot</code>
    <s>Drivers on Duckiebot</s>
    <code>duckiebot</code>

    <code>MODE-offload</code>
    <code>duckiebot</code>
    <s>Drivers on Duckiebot</s>
    <code>laptop</code>

    <code>MODE-bag</code>
    <code>laptop</code>
    <s>Bag file</s>
    <code>laptop</code>
</col4>

<style>
#operation-modes {
font-size: 70%;
}
</style>