Repository Init Content
=======================
 JBPM7.X Process Example

Sample payload to start JBPM Process

http://localhost:8080/kie-server/services/rest/server/containers/claimsSampleProcess_1.0.0/processes/ClaimsSampleProcess.claims-sam-process/instances

{
    
    "claims": {
        "org.zsample.claimssampleprocess.Claims": {
        "claimAmount" : 5000,
        "customerName" : "Diptiman",
		"reason" : "accident"
        }
    }
}


