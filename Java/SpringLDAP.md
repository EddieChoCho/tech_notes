# Spring LDAP

## [Lightweight Directory Access Protocol(LDAP)](https://github.com/EddieChoCho/tech_notes/blob/master/Auth/LDAP.md)

## Packaging Overview
* At a minimum, to use Spring LDAP you need the following:
    * spring-ldap-core: The Spring LDAP library
    * spring-core: Miscellaneous utility classes used internally by the framework   
    * spring-beans: Interfaces and classes for manipulating Java beans
    * spring-data-commons: Base infrastructure for repository suppport and so on
    * slf4j: A simple logging facade, used internally

### ContextSource
#### Base
* You can supply a base LDAP path to the ContextSource, specifying the root in the LDAP tree to which all operations are relative. This means that you are working only with relative distinguished names throughout your system, which is typically rather handy.
* This can significantly simplify working against the LDAP tree. However, there are several occasions when you need to have access to the base path.
    *  One example would be when working with LDAP groups (for example, the groupOfNames object class). 
        * In that case, each group member attribute value needs to be the full DN of the referenced member.      
* For that reason, Spring LDAP has a mechanism by which any Spring-controlled bean may be supplied with the base path on startup. 
* For beans to be notified of the base path, two things need to be in place. 
    * First, the bean that wants the base path reference needs to implement the BaseLdapNameAware interface. 
    * Second, you need to define a BaseLdapPathBeanPostProcessor

### LdapTemplate
* Executes core LDAP functionality and helps to avoid common errors, relieving the user of the burden of looking up contexts, looping through NamingEnumerations and closing contexts.

#### Adding Data
1. Create an instance of `dn`. (e.g. Name instance created through LdapNameBuilder)
2. Use `dn` to create DirContextAdapter instance and set up attributes and the objectClass.
3. Add data through `LdapTemplate.bind` API. 

#### Modify Attributes
1. Create an instance of `dn`.
2. Using `dn` to get the DirContextOperations of the data through LdapTemplate.lookupContext` API.
3. Modify the attributes of DirContextOperations.
3. Execute the modification to LDAP through `LdapTemplate.modifyAttributes` API;

#### Modify DN
1. Create an instance of old `dn`.
2. Create an instance of new `dn`.
3. Rename the `dn` through `LdapTemplate.rename` API with two `dn` instances.
4. The attributes of other data with related to the old `dn` also need to be modified.

#### Removing Data
1. Create an instance of `dn`.
2. Remove the data through `LdapTemplate.unbind` API with `dn` instance.
3. The attributes of other data with related to the removed data's `dn` also need to be removed.

### LdapNameBuilder
* Helper class for building LdapName instances. 
* Note that the first part of a Distinguished Name is the least significant, which means that when adding components, they will be added to the beginning of the resulting string.
* e.g. 
    * LdapNameBuilder.newInstance("dc=261consulting,dc=com").add("ou=people").build().toString();
    * will result in ou=people,dc=261consulting,dc=com.
    
## References
* [Spring LDAP Reference](https://docs.spring.io/spring-ldap/docs/current/reference/)