# Introduction
This script uses the [PMapper code](https://github.com/nccgroup/PMapper#installation-from-source-code) to perform the [cross-account authorization checks](https://github.com/nccgroup/PMapper/wiki/Frequently-Asked-Questions#how-do-i-do-cross-account-authorization-checks).

# Gather Graph Data

```
#authenticate to the organization account in the cli

#if you're using organizations, create the orgs graph
pmapper orgs create

#authenticate to the initial account in the cli

#create the initial account graph
pmapper graph create

#create the second account graph
pmapper graph create

```

# Search Authorization Across Accounts

[action and resource parameters](https://github.com/nccgroup/PMapper/wiki/CLI-Reference#argquery) 

```
import os
import os.path

from principalmapper.common import Graph
from principalmapper.common import Policy
from principalmapper.graphing import gathering, graph_actions
from principalmapper.graphing.cross_account_edges import get_edges_between_graphs
from principalmapper.querying.query_interface import search_authorization_for, search_authorization_across_accounts
from principalmapper.querying import query_utils
from principalmapper.util.storage import get_default_graph_path


#enter the action and resource parameters here:
SOURCE_AWS_ACCOUNT_ID = ""
DEST_AWS_ACCOUNT_ID = ""
ACTION = ""
RESOURCE_ARN = ""
INCLUDE_RESOURCE_POLICY = False

root_graph = graph_actions.get_graph_from_disk(os.path.join(get_default_graph_path(SOURCE_AWS_ACCOUNT_ID)))
prod_graph = graph_actions.get_graph_from_disk(os.path.join(get_default_graph_path(DEST_AWS_ACCOUNT_ID)))

edges = get_edges_between_graphs(root_graph, prod_graph)

for node in root_graph.nodes:
    if INCLUDE_RESOURCE_POLICY:
        resource_policy = query_utils.pull_cached_resource_policy_by_arn(
                    prod_graph,
                    arn=RESOURCE_ARN,
                    query=None
                )
        if isinstance(resource_policy, Policy):
            resource_policy = resource_policy.policy_doc
        result = search_authorization_across_accounts([(root_graph, []), (prod_graph, [])], edges, node, ACTION, RESOURCE_ARN, {}, resource_policy, DEST_AWS_ACCOUNT_ID)
    else:
        result = search_authorization_across_accounts([(root_graph, []), (prod_graph, [])], edges, node, ACTION, RESOURCE_ARN, {})

    if result.allowed == True:
        result.print_result(ACTION, RESOURCE_ARN)
```
