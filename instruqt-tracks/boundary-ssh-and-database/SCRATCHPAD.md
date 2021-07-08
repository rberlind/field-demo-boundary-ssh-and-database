- slug: configure-and-connect-to-your-first-target-with-ssh
  id: wm0enxxguhie
  type: challenge
  title: Configure Your First Host and Connect with SSH
  teaser: Use Boundary to SSH into a target host.
  notes:
  - type: text
    contents: TODO
  assignment: |-
    *Configure Your First Host and Connect with SSH*

    In the admin console, you saw that `localhost` was listed as *Generated target* with connection type, TCP. The
    default target is a TCP target with a default port of `22` (SSH). In this tutorial, you will initiate an `ssh`
    session to this default target using the CLI .

    Before you authenticate, you'll need the generated auth method, username and password.  You can see those values
    in the environment on the Controller. Run the below commands in the Controller tab.

    ```
    echo "AUTH METHOD ID:" $GEN_AUTH_METHOD
    echo "ADMIN USERNAME:" $ADMIN_USERNAME
    echo "ADMIN PASSWORD:" $ADMIN_PASSWORD
    ```

    _TODO: echo all the below exports into the environment?_

    You should see the values you will be using, log in using those environment variables.

    ```
    boundary authenticate password \
      -auth-method-id=$GEN_AUTH_METHOD \
      -keyring-type="none"\
      -login-name=$ADMIN_USERNAME \
      -password=$ADMIN_PASSWORD
    ```

    You should see output similar to the below.

    ```
    Authentication information:
      Account ID:      acctpw_3po3pAsJBp
      Auth Method ID:  ampw_nqedpwu2Vh
      Expiration Time: Wed, 07 Jul 2021 00:43:12 UTC
      User ID:         u_pNPr4pFTTz

    Storing the token in a keyring was disabled. The token is:
    at_oRx...truncated...j6i
    ...
    ```

    You can see a Boundary token returned. You want to store that value in an environment variable.

    ```
    export BOUNDARY_TOKEN="<TOKEN_FROM_ABOVE>"
    ```

    For a simple one-liner to to store that value in the environment for later usage, use the below command.

    ```
    export BOUNDARY_TOKEN=$( \
      boundary authenticate password \
      -format=json \
      -auth-method-id=$GEN_AUTH_METHOD \
      -keyring-type="none"\
      -login-name=$ADMIN_USERNAME \
      -password=$ADMIN_PASSWORD | jq -r .item.attributes.token)
    ```

    Now that you are authenticated, you need to get the project scope ID so you can see it's host sets and hosts.  You
    will use the generated project scope for this challenge. List the scopes currently in Boundary.

    ```
    boundary scopes list
    ```

    You should output similar to the below text. Note that it is an *org scope*.

    ```
    Scope information:
      ID:                    o_fCudeW87DE
        Version:             1
        Name:                Generated org scope
        Description:         Provides an initial org scope in Boundary
        Authorized Actions:
          no-op
          read
          update
          delete
    ```

    This returns your *org* scopes. You need the *project* scope, so you will need to run another `scopes list`
    command, but this time, with the org scope ID passed in as an argument.

    ```
    boundary scopes list -scope-id=<ORG_SCOPE_ID_FROM_ABOVE>
    ```

    You should see output like the below. Note this time you are getting a *project scope*, not an org scope.

    ```
    Scope information:
      ID:                    p_N4JvIZWqZc
        Version:             1
        Name:                Generated project scope
        Description:         Provides an initial project scope in Boundary
        Authorized Actions:
          no-op
          read
          update
          delete
    ```

    An easy one liner to store both the global org scope ID as well as the project scope ID in the environment would
    look like the below.

    ```
    export GEN_GLOBAL_SCOPE_ID=$(boundary scopes list -format=json | jq -r '.items[0].id')
    export GEN_PROJ_SCOPE_ID=$(boundary scopes list -scope-id=$GEN_GLOBAL_SCOPE_ID -format=json | jq -r '.items[0].id')
    ```

    With your project scope ID in hand, you can now look up the targets associated with the scope. You can paste the
    scope ID from above in the below command.

    ```
    boundary targets list -scope-id=<PROJECT_SCOPE_ID_FROM_ABOVE>
    ```

    Or you can use the project scope ID you stored in the environment to make it a little easier.

    ```
    boundary targets list -scope-id=$GEN_PROJ_SCOPE_ID
    ```

    You should see output like the below, including the target ID. This is the ID of the target that you will connect
    to.

    ```
    Target information:
      ID:                    ttcp_s3ngxT2ofL
        Version:             1
        Type:                tcp
        Name:                Generated target
        Description:         Provides an initial target in Boundary
        Authorized Actions:
          no-op
          read
          update
          delete
          add-host-sets
          set-host-sets
          remove-host-sets
          authorize-session
    ```

    Again, a simple one-liner to store the target ID in the environment.

    ```
    export GEN_TARGET_ID=$(boundary targets list -scope-id=$GEN_PROJ_SCOPE_ID -format=json | jq -r '.items[0].id')
    ```

    You can view the details of the target using the `targets read` command. An example:

    ```
    boundary targets read -id=<TARGET_ID_FROM_ABOVE>
    ```

    Or, using the value stored in the environment.

    ```
    boundary targets read -id=$GEN_TARGET_ID
    ```

    You should see output like the below.

    ```
    Target information:
      Created Time:               Wed, 30 Jun 2021 00:37:51 UTC
      Description:                Provides an initial target in Boundary
      ID:                         ttcp_s3ngxT2ofL
      Name:                       Generated target
      Session Connection Limit:   -1
      Session Max Seconds:        28800
      Type:                       tcp
      Updated Time:               Wed, 30 Jun 2021 00:37:51 UTC
      Version:                    1

      Scope:
        ID:                       p_N4JvIZWqZc
        Name:                     Generated project scope
        Parent Scope ID:          o_fCudeW87DE
        Type:                     project

      Authorized Actions:
        no-op
        read
        update
        delete
        add-host-sets
        set-host-sets
        remove-host-sets
        authorize-session

      Host Sets:
        Host Catalog ID:          hcst_jLHX9jl2kL
        ID:                       hsst_4eg3luVSyC

      Attributes:
        Default Port:             22
    ```

    You are ready to connect to the target. You can paste the target ID into the below command to connect.

    ```
    boundary connect ssh -target-id=<TARGET_ID_FROM_ABOVE>
    ```

    Or, you can use the target ID you stored in the environment.

    ```
    boundary connect ssh -target-id=$GEN_TARGET_ID
    ```

    Be sure to respond `yes` when asked if you are sure you want to continue connecting.

    _TODO: add a diagram of the ref architecture and components to the notes and tell the user they can go view that if they need a refresher._


    You have now successfully connected to your first target using Boundary!
  tabs:
  - title: VS Code
    type: service
    hostname: boundary-controller
    port: 8443
  - title: Controller
    type: terminal
    hostname: boundary-controller
  - title: Worker
    type: terminal
    hostname: boundary-worker
  - title: Boundary UI
    type: service
    hostname: boundary-controller
    port: 9200
  difficulty: advanced
  timelimit: 3600
- slug: configure-a-new-host-set-connect-postgres
  id: 1ut7x9mfbihc
  type: challenge
  title: Configure a New Host Set and Connect to a Postgres Host
  teaser: TODO
  notes:
  - type: text
    contents: TODO
  assignment: |-
    # Configure a New Host Set and Connect to a Postgres Host

    # Create an Org

    ```
    boundary scopes create \
      -scope-id=global \
      -name=HashiCups_IT \
      -description="HashiCups IT Team"
    ```

    You should see some output like the below.

    ```
    Scope information:
      Created Time:        Fri, 02 Jul 2021 21:22:22 UTC
      Description:         HashiCups IT Team
      ID:                  o_tdsUUNNHdn
      Name:                HashiCups_IT
      Updated Time:        Fri, 02 Jul 2021 21:22:22 UTC
      Version:             1

      Scope (parent):
        ID:                global
        Name:              global
        Type:              global

      Authorized Actions:
        no-op
        read
        update
        delete
    ```

    List your scopes to confirm it has been created.

    ```
    boundary scopes list
    ```

    You should see some output like the below.

    ```
    Scope information:
      ID:                    o_FnOiDQDaT8
        Version:             1
        Name:                Generated org scope
        Description:         Provides an initial org scope in Boundary
        Authorized Actions:
          no-op
          read
          update
          delete

      ID:                    o_tdsUUNNHdn
        Version:             1
        Name:                HashiCups_IT
        Description:         HashiCups IT Team
        Authorized Actions:
          no-op
          read
          update
          delete
      ```

    Then put the newly created org ID into the environment.

    ```
    export ORG_ID=<ORG_ID_FROM_ABOVE>
    ```

    Or you can use this simple one-liner to set the new `ORG_ID`.

    ```
    export ORG_ID=$(boundary scopes list -format=json | jq -r '.items[-1].id')
    ```

    # Create a Project

    ```
    boundary scopes create \
      -scope-id=$ORG_ID \
      -name=HashiCups_QA_Tests \
      -description="Manage QA machines"
    ```

    You should see some output like the below.

    ```
    Scope information:
      Created Time:        Fri, 02 Jul 2021 21:26:08 UTC
      Description:         Manage QA machines
      ID:                  p_FzsymjnX3r
      Name:                HashiCups_QA_Tests
      Updated Time:        Fri, 02 Jul 2021 21:26:08 UTC
      Version:             1

      Scope (parent):
        ID:                o_tdsUUNNHdn
        Name:              HashiCups_IT
        Parent Scope ID:   global
        Type:              org

      Authorized Actions:
        no-op
        read
        update
        delete
    ```

    List the project under the HashiCups_IT org to verify it was created correctly.

    ```
    boundary scopes list -scope-id=$ORG_ID
    ```

    You should see some output like the below.

    ```
    Scope information:
      ID:                    p_FzsymjnX3r
        Version:             1
        Name:                HashiCups_QA_Tests
        Description:         Manage QA machines
        Authorized Actions:
          no-op
          read
          update
          delete
    ```

    Then put the newly created project ID into the environment.

    ```
    export PROJECT_ID=<PROJECT_ID_FROM_ABOVE>
    ```

    Or you can use this simple one-liner to set the new `PROJECT_ID`.

    ```
    export PROJECT_ID=$(boundary scopes list -scope-id=$ORG_ID -format=json | jq -r '.items[0].id')
    ```

    ---

    Now that you have created a new org and project, you can configure the new hosts that you will connect to. All
    hosts must be part of a host set, both of which must be within a host catalog, so you need to create a host catalog
    first.

    ```
    boundary host-catalogs create static -scope-id=$PROJECT_ID -name=DevOps \
        -description="For DevOps usage"
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    Then put the newly created host catalog ID into the environment.

    ```
    export HOST_CATALOG_ID=<HOST_CATALOG_ID_FROM_ABOVE>
    ```

    TODO: one-liner

    Now, create a new host named, "postgres" with description, "Postgres host" under the newly created host catalog.

    ```
    boundary hosts create static -name=postgres -description="Postgres host" \
    -address="127.0.0.1" -host-catalog-id=$HOST_CATALOG_ID
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    Repeat the step to create another host named, `localhost`.

    ```
    boundary hosts create static -name=localhost -description="Localhost for testing" \
      -address="localhost" -host-catalog-id=$HOST_CATALOG_ID
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    ```
    boundary host-sets create static -name="test-machines" \
    -description="Test machine host set" -host-catalog-id=$HOST_CATALOG_ID
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    ```
    boundary hosts list -host-catalog-id=$HOST_CATALOG_ID
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    ```
    boundary host-sets add-hosts -id=<host_set_id> -host=<postgres_host_id> \
      -host=<localhost_host_id>
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    ```
    boundary targets create tcp -name="tests" -description="Test target" \
        -default-port=22 -scope-id=$PROJECT_ID -session-connection-limit="-1"
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    ```
    boundary targets add-host-sets -id=<target_id> -host-set=<host_set_id>
    ```

    You should see some output like the below.

    ```
    TODO
    ```

    Connect to it...
  tabs:
  - title: VS Code
    type: service
    hostname: boundary-controller
    port: 8443
  - title: Controller
    type: terminal
    hostname: boundary-controller
  - title: Worker
    type: terminal
    hostname: boundary-worker
  - title: Boundary UI
    type: service
    hostname: boundary-controller
    port: 9200
  difficulty: advanced
  timelimit: 3600
- slug: configure-users-and-groups
  id: niaptwmxppym
  type: challenge
  title: Configure Users and Groups
  teaser: TODO
  notes:
  - type: text
    contents: TODO
  assignment: |-
    '*Configure Users and Groups*'

    ```
    boundary auth-methods create password -scope-id=$ORG_ID \
      -name="hashicups_auth_method" \
      -description="HashiCups auth method"
    ```

    ```
    boundary accounts create password \
      -auth-method-id=<AUTH_METHOD_ID_FROM_ABOVE> \
      -login-name="hashicups_admin" \
      -password="password" \
      -name=hashicups_admin \
      -description="HashiCups Admin Demo Account"
    ```

    You should see output like the below.

    ```
    TODO
    ```

    ```
    boundary users create -name="tester01" -description="A test user" -scope-id=$ORG_ID
    ```

    You should see output like the below.

    ```
    TODO
    ```

    ```
    boundary users set-accounts -id=<user_id> -account=<account_id>
    ```

    You should see output like the below.

    ```
    TODO
    ```

    ```
    boundary authenicate password -login-name="tester01" -password="supersecure" \
    -auth-method-id=<auth_method_id>
    ```

    You should see output like the below.

    ```
    TODO
    ```

    ```
    boundary authenticate password -auth-method-id=ampw_1234567890 \
    -login-name=admin -password=password
    ```

    You should see output like the below.

    ```
    TODO
    ```

    ```
    boundary groups create -name="group01" -description="A test group" -scope-id=$ORG_ID
    ```

    You should see output like the below.

    ```
    TODO
    ```

    ```
    boundary groups add-members -id=<group_id> -member=<user_id>
    ```

    You should see output like the below.

    ```
    TODO
    ```
  difficulty: advanced
  timelimit: 600
- slug: explore-the-admin-ui
  id: wyaxsa25ysd1
  type: challenge
  title: Explore the Admin UI
  teaser: Get familiar with Boundary's concepts and admin UI before jumping into accessing targets.
  notes:
  - type: text
    contents: |-
      Boundary enables simple and secure access to dynamic infrastructure by:

      - Identity-based access controls: Streamline just-in-time access to privileged sessions (e.g. TCP, SSH, RDP) for users and applications. Tightly control access permissions with extensible role-based access controls.
      - Access Automation: Define your perimeter of resources, identities, and access controls as code through Boundary's fully-instrumented Terraform Provider, REST API, CLI, and SDK. Automate the discovery of new resources and enforcement of existing policies as resources are provisioned.
      - Session Visibility: Security administrators gain monitor and managed the privileged sessions established with Boundary. Export session logs to your analytics tool of choice.

      In this track, you will learn how to use Boundary to dynamically access hosts via SSH, as well as remote Postgres database access.
  - type: text
    contents: Traditional approaches like SSH bastion hosts or VPNs that require distributing and managing credentials, configure network controls like firewalls, and expose the private network. Boundary provides a secure way to access to hosts and critical systems without having to manage credentials or expose your network.
  - type: text
    contents: "| Resource | Description |\n| --- | --- |\n| Scope\t| Abstract permission boundary modeled as a container. A scope can contain scopes forming a tree. |\n| Organization | Top-level container (scope) which owns zero to many projects and zero to many authentication methods. An organization inherits from scope allowing it to own zero to many groups, roles, policies, targets, host catalogs or credential stores. |\n| Project\t| Child scope of an organization. |\n| User | Any entity authorized to access Boundary using authentication credentials specific to one of the configured authentication methods. A user can belong to zero or more groups. |\n| Group\t| Collection of users used for access control. A group is owned by one and only one scope. |\n| Role | Collection of capabilities granted to any principal (user, group, or project) the role is assigned to. A role belongs to one and only one scope, and owns zero or more direct grants |\n| Host | Computing element with a network address reachable from Boundary. |\n| Host catalog | Permission boundary modeled as a container containing scopes forming a tree. |\n| Host set |Subset of hosts from the set of hosts of the host catalog it belongs to. A host set belongs to one and only one host; therefore, it gets deleted when its host catalog is deleted. |\n| Target | Networked service a user can connect to and interact with through Boundary. A target can contain zero or more host sets. |"
  - type: text
    contents: '''TODO: Explain Controllers vs Workers and show/ reference architecture.'''
  assignment: |-
    # Explore the Admin Console

    Boundary's Admin Console provides an easy way to manage resources. Before using Boundary to securely access any
    dynamic infrastructure, you should take a tour of the console to understand all the components.

    TODO: update or remove this based on the direction of the track.

    In this track, Boundary has already been
    [installed](https://learn.hashicorp.com/tutorials/boundary/getting-started-install?in=boundary/getting-started)
    for you. A Boundary Controller and a Boundary Worker are also already running. To make your initial tour through
    Boundary a little more interesting, Boundary was
    [initialized with generated resources](https://www.boundaryproject.io/docs/installing/no-gen-resources).

    What are generated resources? Boundary can automatically generate a number of resources to make getting started
    easier. Default scopes, auth methods, user, account, and targets are just some of the resources Boundary will
    generate unless you tell it not to.

    If you are not familiar with installing Boundary, the reference architecture, or concepts like scopes, auth methods,
    users, accounts, and more - check out the [Install and Configure Boundary Track](TODO).

    ---

    # Exploring Roles, Principals and Grants in the Admin UI

    Open the Boundary UI tab. You should see a login screen. To access the password for the `admin` user that was
    generated for you, grab it from the environment in the Controller terminal tab.

    ```
    echo $ADMIN_PASSWORD
    ```

    Once you have logged in, you will see the home screen containing all the top level resources in Boundary. Select
    *Roles* in the left sidebar, then select the "Administration" role.

    Notice that `admin` user is listed. *User*, *group*, and *project* are a type of principal which can be assigned to
    roles.

    Click on the *Grants* tab to view the permissions allowed on this role.  Grants represent strings of actions on
    resources: `id=<resource_id>; action=<actions>`

    The grant for Administration role indicates that all actions (`actions=*`) on all resources (`id=*;type=*`) are
    permitted. Refer to the [documentation](https://www.boundaryproject.io/docs/concepts/security/permissions#permission-grant-formats)
    for more details.

    Return to the *Roles* list and select *Login and Default Grants* role.  Click *Grants* to view its permissions.

    A role can have multiple grants defined. Those grants are deleted when the role is deleted. A grant is also deleted
    if its associated resource is deleted.

    Select *Projects* and then *Generated project scope*. Notice that you can see *Sessions*, *Targets* and
    *Host Catalogs*.

    Select *Host Catalogs -> Generated host catalog -> Host Sets -> Generated host set* to view the details of the
    *Host Set*. You can then select the *Hosts* tab to view attached hosts.

    ---

    Now that you have taken a tour of the admin UI and understand the generated resources, you are ready to connect
    to your first target via Boundary.