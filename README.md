Act as an expert software QA automation engineer specialized in OpenText ALM Octane REST API integrations and Robot Framework.

### Context
I am executing manual test runs via REST API in ALM Octane using Robot Framework. The test run is successfully created and appears under the Test entity, but the associated User Story in the Backlog view still shows no test runs / blank status in "Last runs".

### Payload Currently Used
{
  "data": [
    {
      "name": "Name_of_Run",
      "test": {
        "type": "test",
        "id": "892028"
      },
      "native_status": {
        "type": "list_node",
        "id": "list_node.run_native_status.passed"
      },
      "release": {
        "type": "release",
        "id": "Release_id"
      },
      "taxonomies": {
        "data": [
          {
            "type": "taxonomy_node",
            "id": "ENV_id",
            "subtype": "taxonomy_item_node"
          }
        ]
      },
      "user_tags": {
        "data": [
          {
            "type": "user_tag",
            "id": "Tag_id"
          }
        ]
      },
      "run_by": {
        "type": "workspace_user",
        "id": "Run_User_id"
      }
    }
  ]
}

### Problem
In the ALM Octane Backlog grid, the User Story shows "Test Coverage: 4 tests", but the "Last runs" column does not display the execution status. 

### What I Need
1. Explain the exact REST API schema additions needed in the payload to link the run directly to the User Story (`work_item` / `requirement`) and current `sprint`.
2. Provide an updated JSON body template with all necessary relationship fields included.
3. Provide a Python or Robot Framework code snippet to query the User Story ID linked to a Test ID via Octane REST API before posting the run.
