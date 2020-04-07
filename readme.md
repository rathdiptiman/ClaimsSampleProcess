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


package com.wmt.util;

import java.io.FileInputStream;
import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.Hashtable;
import java.util.Properties;

import javax.naming.AuthenticationException;
import javax.naming.AuthenticationNotSupportedException;
import javax.naming.Context;
import javax.naming.NameNotFoundException;
import javax.naming.NamingEnumeration;
import javax.naming.NamingException;
import javax.naming.SizeLimitExceededException;
import javax.naming.directory.Attribute;
import javax.naming.directory.Attributes;
import javax.naming.directory.DirContext;
import javax.naming.directory.InitialDirContext;
import javax.naming.directory.SearchControls;
import javax.naming.directory.SearchResult;

import org.apache.commons.logging.Log;
import org.apache.commons.logging.LogFactory;



public class LDAPUtil {
	
	private static final String INITIAL_CONTEXT_FACTORY="com.sun.jndi.ldap.LdapCtxFactory";
	private static final String PROVIDER_URL=System.getProperty("LDAP_URL");
	private static final String REFERRAL="follow";
	private static final String SECURITY_AUTHENTICATION="simple";
	private static final String SECURITY_PRINCIPAL=System.getProperty("LDAP_PRINCIPAL");
	private static final String SECURITY_CREDENTIALS=System.getProperty("LDAP__CREDENTIALS");
	

	
	private static Log LOGGER = LogFactory.getLog(LDAPUtil.class);
	public static Boolean validateLogin(String userName, String userPassword) {
	    Hashtable<String, String> env = new Hashtable<String, String>();
	    DirContext ctx;
	    
	    try {
		    
	    LOGGER.info (" PROVIDER_URL :: " + PROVIDER_URL);
	    LOGGER.info (" SECURITY_PRINCIPAL :: " + SECURITY_PRINCIPAL);
	    LOGGER.info (" SECURITY_CREDENTIALS :: " + SECURITY_CREDENTIALS);
	    
	    LOGGER.info (" PROVIDER_URL :: " + System.getProperty("LDAP_URL") );
	    LOGGER.info (" SECURITY_PRINCIPAL :: " + System.getProperty("LDAP_PRINCIPAL"));
	    LOGGER.info (" SECURITY_CREDENTIALS :: " + System.getProperty("LDAP__CREDENTIALS"));
	    LOGGER.info (" SECURITY_SEARCH :: " + System.getProperty("LDAP__SEARCH"));
	    
	    env.put(Context.INITIAL_CONTEXT_FACTORY, INITIAL_CONTEXT_FACTORY);
	    env.put(Context.PROVIDER_URL, PROVIDER_URL);

	    // To get rid of the PartialResultException when using Active Directory
	    env.put(Context.REFERRAL, REFERRAL);

	    // Needed for the Bind (User Authorized to Query the LDAP server) 
	    env.put(Context.SECURITY_AUTHENTICATION, SECURITY_AUTHENTICATION);
	    //env.put(Context.SECURITY_PRINCIPAL, System.getProperty("LDAP_PRINCIPAL") );
	    env.put(Context.SECURITY_PRINCIPAL, "CN=jbpmprod\\, jbpmprod,OU=Service Accounts,OU=Cognizant,DC=us,DC=wmgi,DC=com");
	    env.put(Context.SECURITY_CREDENTIALS,  SECURITY_CREDENTIALS);

	   
	       ctx = new InitialDirContext(env);
	    } catch (NamingException e) {
	    	e.printStackTrace();
	       throw new RuntimeException(e);
	    }

	    NamingEnumeration<SearchResult> results = null;

	    try {
	       SearchControls controls = new SearchControls();
	       controls.setSearchScope(SearchControls.SUBTREE_SCOPE); // Search Entire Subtree
	       controls.setCountLimit(1);   //Sets the maximum number of entries to be returned as a result of the search
	       controls.setTimeLimit(5000); // Sets the time limit of these SearchControls in milliseconds

	       String searchString = "(&(objectCategory=user)(sAMAccountName=" + userName + "))";

	       results = ctx.search( System.getProperty("LDAP__SEARCH"), searchString, controls);

	       if (results.hasMore()) {

	           SearchResult result = (SearchResult) results.next();
	           Attributes attrs = result.getAttributes();
	           Attribute dnAttr = attrs.get("distinguishedName");
	           String dn = (String) dnAttr.get();

	           // User Exists, Validate the Password

	           env.put(Context.SECURITY_PRINCIPAL, dn);
	           env.put(Context.SECURITY_CREDENTIALS, userPassword);

	           new InitialDirContext(env); // Exception will be thrown on Invalid case
	           return true;
	       } 
	       else 
	           return false;

	    } catch (AuthenticationException e) { // Invalid Login
	    	LOGGER.info (PrintStackTraceToString(e));
	    	
	        return false;
	    } catch (NameNotFoundException e) { // The base context was not found.
	    	LOGGER.info (PrintStackTraceToString(e));
	        return false;
	    } catch (SizeLimitExceededException e) {
	    	LOGGER.info ("LDAP Query Limit Exceeded, adjust the query to bring back less records \n"+PrintStackTraceToString(e) );
	    	return false;
	    } catch (NamingException e) {
	    	LOGGER.info (PrintStackTraceToString(e));
	    	return false;
	    } catch (Exception e) {
	    	LOGGER.info (PrintStackTraceToString(e));
	    	return false;
	    }
	    finally {

	       if (results != null) {
	          try { results.close(); } catch (Exception e) { /* Do Nothing */ }
	       }

	       if (ctx != null) {
	          try { ctx.close(); } catch (Exception e) { /* Do Nothing */ }
	       }
	    }
	}
	private static String PrintStackTraceToString(Exception e) {
		// TODO Auto-generated method stub
		StringWriter errors = new StringWriter();
		if(e!=null)
		e.printStackTrace(new PrintWriter(errors));
		return errors.toString();
	}
}

