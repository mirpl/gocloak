# gocloak
[![codebeat badge](https://codebeat.co/badges/c699bc56-aa5f-4cf5-893f-5cf564391b94)](https://codebeat.co/projects/github-com-nerzal-gocloak-master)
[![Go Report Card](https://goreportcard.com/badge/github.com/Nerzal/gocloak)](https://goreportcard.com/report/github.com/Nerzal/gocloak)
[![Go Doc](https://godoc.org/github.com/Nerzal/gocloak?status.svg)](https://godoc.org/github.com/Nerzal/gocloak)
[![Build Status](https://github.com/Nerzal/gocloak/workflows/Tests/badge.svg)](https://github.com/Nerzal/gocloak/actions?query=branch%3Amaster+event%3Apush)
[![GitHub release](https://img.shields.io/github/tag/Nerzal/gocloak.svg)](https://GitHub.com/Nerzal/gocloak/releases/)
[![codecov](https://codecov.io/gh/Nerzal/gocloak/branch/master/graph/badge.svg)](https://codecov.io/gh/Nerzal/gocloak)
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2FNerzal%2Fgocloak.svg?type=shield)](https://app.fossa.io/projects/git%2Bgithub.com%2FNerzal%2Fgocloak?ref=badge_shield)


Golang Keycloak API Package

This client is based on: [go-keycloak](https://github.com/PhilippHeuer/go-keycloak)

For Questions either raise an issue, or come to the [gopher-slack](https://invite.slack.golangbridge.org/) into the channel [#gocloak](https://gophers.slack.com/app_redirect?channel=gocloak)

If u are using the echo framework have a look at [gocloak-echo](https://github.com/Nerzal/gocloak-echo)

Benchmarks: https://nerzal.github.io/gocloak/dev/bench/


## Contribution
(WIP) https://github.com/Nerzal/gocloak/wiki/Contribute

## Changelog

### v5:
There is only one change, but it's backward incompatible:
* Wrap Errors and use APIError struct to also provide the httpstatus code. ([#146](https://github.com/Nerzal/gocloak/pull/146))

### v4:
There are a lot of backward incompatible changes:
* all functions what create an object now return an ID of the created object. The return statement of those functions has been changed from (error) to (string, error)
* All structures now use pointers instead of general types (bool -> *bool, string -> *string). It has been done to properly use omitempty tag, otherwise it was impossible to set a false value for any of the bool propertires.

## Usage



### Importing

```go
	import "github.com/Nerzal/gocloak/v5"
```

or v3 (latest release is v3.10.0):

```go
	import "github.com/Nerzal/gocloak/v3"
```

### Create New User
```go
	client := gocloak.NewClient("https://mycool.keycloak.instance")
	token, err := client.LoginAdmin("user", "password", "realmName")
	if err != nil {
		panic("Something wrong with the credentials or url")
	}
	user := gocloak.User{
		FirstName: "Bob",
		LastName:  "Uncle",
		Email:     "something@really.wrong",
		Enabled:   true,
		Username:  "CoolGuy",
	}
	_, err = client.CreateUser(token.AccessToken, "realm", user)
	if err != nil {
		panic("Oh no!, failed to create user :(")
	}
```

### Introspect Token
```go
	client := gocloak.NewClient(hostname)
	token, err := client.LoginClient(clientid, clientSecret, realm)
	if err != nil {
		panic("Login failed:"+ err.Error())
	}

	rptResult, err := client.RetrospectToken(token.AccessToken, clientid, clientSecret, realm)
	if err != nil {
		panic("Inspection failed:"+ err.Error())
	}

	if !rptResult.Active {
		panic("Token is not active")
	}

	permissions := rptResult.Permissions
	//Do something with the permissions ;)
```

## Features

```go
// GoCloak holds all methods a client should fullfill
type GoCloak interface {
	GetRequestingPartyToken(token, realm string, options RequestingPartyTokenOptions) (*JWT, error)

	Login(clientID string, clientSecret string, realm string, username string, password string) (*JWT, error)
	Logout(clientID, clientSecret, realm, refreshToken string) error
	LogoutPublicClient(clientID, realm, accessToken, refreshToken string) error
	LoginClient(clientID, clientSecret, realm string) (*JWT, error)
	LoginAdmin(username, password, realm string) (*JWT, error)
	RequestPermission(clientID string, clientSecret string, realm string, username string, password string, permission string) (*JWT, error)
	RefreshToken(refreshToken string, clientID, clientSecret, realm string) (*JWT, error)
	DecodeAccessToken(accessToken string, realm string) (*jwt.Token, *jwt.MapClaims, error)
	DecodeAccessTokenCustomClaims(accessToken string, realm string, claims jwt.Claims) (*jwt.Token, error)
	RetrospectToken(accessToken string, clientID, clientSecret string, realm string) (*RetrospecTokenResult, error)
	GetIssuer(realm string) (*IssuerResponse, error)
	GetCerts(realm string) (*CertResponse, error)
	GetServerInfo(accessToken string) (*ServerInfoRepesentation, error)
	GetUserInfo(accessToken string, realm string) (*UserInfo, error)
	SetPassword(token string, userID string, realm string, password string, temporary bool) error
	ExecuteActionsEmail(token string, realm string, params ExecuteActionsEmail) error

	CreateUser(token string, realm string, user User) (string, error)
	CreateGroup(accessToken string, realm string, group Group) error
	CreateChildGroup(token string, realm string, groupID string, group Group) (string, error)
	CreateClientRole(accessToken string, realm string, clientID string, role Role) error
	CreateClient(accessToken string, realm string, clientID Client) error
	CreateClientScope(accessToken string, realm string, scope ClientScope) error
	CreateComponent(accessToken string, realm string, component Component) error

	UpdateUser(accessToken string, realm string, user User) error
	UpdateGroup(accessToken string, realm string, updatedGroup Group) error
	UpdateRole(accessToken string, realm string, clientID string, role Role) error
	UpdateClient(accessToken string, realm string, updatedClient Client) error
	UpdateClientScope(accessToken string, realm string, scope ClientScope) error

	DeleteUser(accessToken string, realm, userID string) error
	DeleteComponent(accessToken string, realm, componentID string) error
	DeleteGroup(accessToken string, realm, groupID string) error
	DeleteClientRole(accessToken string, realm, clientID, roleName string) error
	DeleteClient(accessToken string, realm, clientID string) error
	DeleteClientScope(accessToken string, realm, scopeID string) error

	GetClient(accessToken string, realm string, clientID string) (*Client, error)
	GetClientsDefaultScopes(token string, realm string, clientID string) ([]*ClientScope, error)
	AddDefaultScopeToClient(token string, realm string, clientID string, scopeID string) error
	RemoveDefaultScopeFromClient(token string, realm string, clientID string, scopeID string) error
	GetClientsOptionalScopes(token string, realm string, clientID string) ([]*ClientScope, error)
	AddOptionalScopeToClient(token string, realm string, clientID string, scopeID string) error
	RemoveOptionalScopeFromClient(token string, realm string, clientID string, scopeID string) error
	GetDefaultOptionalClientScopes(token string, realm string) ([]*ClientScope, error)
	GetDefaultDefaultClientScopes(token string, realm string) ([]*ClientScope, error)
	GetClientScope(token string, realm string, scopeID string) (*ClientScope, error)
	GetClientScopes(token string, realm string) ([]*ClientScope, error)
	GetClientSecret(token string, realm string, clientID string) (*CredentialRepresentation, error)
	GetClientServiceAccount(token string, realm string, clientID string) (*User, error)
	RegenerateClientSecret(token string, realm string, clientID string) (*CredentialRepresentation, error)
	GetKeyStoreConfig(accessToken string, realm string) (*KeyStoreConfig, error)
	GetUserByID(accessToken string, realm string, userID string) (*User, error)
	GetUserCount(accessToken string, realm string) (int, error)
	GetUsers(accessToken string, realm string, params GetUsersParams) ([]*User, error)
	GetUserGroups(accessToken string, realm string, userID string) ([]*UserGroup, error)
	GetComponents(accessToken string, realm string) ([]*Component, error)
	GetGroups(accessToken string, realm string, params GetGroupsParams) ([]*Group, error)
	GetGroup(accessToken string, realm, groupID string) (*Group, error)
	GetDefaultGroups(accessToken string, realm string) ([]*Group, error)
	AddDefaultGroup(accessToken string, realm string, groupID string) error
	RemoveDefaultGroup(accessToken string, realm string, groupID string) error
	GetGroupMembers(accessToken string, realm, groupID string, params GetGroupsParams) ([]*User, error)
	GetRoleMappingByGroupID(accessToken string, realm string, groupID string) (*MappingsRepresentation, error)
	GetRoleMappingByUserID(accessToken string, realm string, userID string) (*MappingsRepresentation, error)
	GetClientRoles(accessToken string, realm string, clientID string) ([]*Role, error)
	GetClientRole(token string, realm string, clientID string, roleName string) (*Role, error)
	GetClients(accessToken string, realm string, params GetClientsParams) ([]*Client, error)
	AddClientRoleComposite(token string, realm string, roleID string, roles []Role) error
	DeleteClientRoleComposite(token string, realm string, roleID string, roles []Role) error
	GetUsersByRoleName(token string, realm string, roleName string) ([]*User, error)
	GetUsersByClientRoleName(token string, realm string, clientID string, roleName string, params GetUsersByRoleParams) ([]*User, error)
	UserAttributeContains(attributes map[string][]string, attribute string, value string) bool
	CreateClientProtocolMapper(token, realm, clientID string, mapper ProtocolMapperRepresentation) error
	UpdateClientProtocolMapper(token, realm, clientID string, mapperID string, mapper ProtocolMapperRepresentation) error
	DeleteClientProtocolMapper(token, realm, clientID, mapperID string) error

	// *** Realm Roles ***

	CreateRealmRole(token string, realm string, role Role) error
	GetRealmRole(token string, realm string, roleName string) (*Role, error)
	GetRealmRoles(accessToken string, realm string) ([]*Role, error)
	GetRealmRolesByUserID(accessToken string, realm string, userID string) ([]*Role, error)
	GetRealmRolesByGroupID(accessToken string, realm string, groupID string) ([]*Role, error)
	UpdateRealmRole(token string, realm string, roleName string, role Role) error
	DeleteRealmRole(token string, realm string, roleName string) error
	AddRealmRoleToUser(token string, realm string, userID string, roles []Role) error
	DeleteRealmRoleFromUser(token string, realm string, userID string, roles []Role) error
	AddRealmRoleToGroup(token string, realm string, groupID string, roles []Role) error
	DeleteRealmRoleFromGroup(token string, realm string, groupID string, roles []Role) error
	AddRealmRoleComposite(token string, realm string, roleName string, roles []Role) error
	DeleteRealmRoleComposite(token string, realm string, roleName string, roles []Role) error


	// *** Client Roles ***

	AddClientRoleToGroup(token string, realm string, clientID string, groupID string, roles []Role) error
	DeleteClientRoleFromGroup(token string, realm string, clientID string, groupID string, roles []Role) error
	GetCompositeClientRolesByRoleID(token string, realm string, clientID string, roleID string) ([]*Role, error)
	GetClientRolesByUserID(token string, realm string, clientID string, userID string) ([]*Role, error)
	GetClientRolesByGroupID(token string, realm string, clientID string, groupID string) ([]*Role, error)
	GetCompositeClientRolesByUserID(token string, realm string, clientID string, userID string) ([]*Role, error)
	GetCompositeClientRolesByGroupID(token string, realm string, clientID string, groupID string) ([]*Role, error)

	// *** Realm ***

	GetRealm(token string, realm string) (*RealmRepresentation, error)
	GetRealms(token string) ([]*RealmRepresentation, error)
	CreateRealm(token string, realm RealmRepresentation) (string, error)
	UpdateRealm(token string, realm RealmRepresentation) error
	DeleteRealm(token string, realm string) error
	ClearRealmCache(token string, realm string) error
	ClearUserCache(token string, realm string) error
	ClearKeysCache(token string, realm string) error

	GetClientUserSessions(token, realm, clientID string) ([]*UserSessionRepresentation, error)
	GetClientOfflineSessions(token, realm, clientID string) ([]*UserSessionRepresentation, error)
	GetUserSessions(token, realm, userID string) ([]*UserSessionRepresentation, error)
	GetUserOfflineSessionsForClient(token, realm, userID, clientID string) ([]*UserSessionRepresentation, error)

	// *** Protection API ***
	GetResource(token string, realm string, clientID string, resourceID string) (*Resource, error)
	GetResources(token string, realm string, clientID string) ([]*Resource, error)
	CreateResource(token string, realm string, clientID string, resource Resource) (*Resource, error)
	UpdateResource(token string, realm string, clientID string, resource Resource) error
	DeleteResource(token string, realm string, clientID string, resourceID string) error

	GetScope(token string, realm string, clientID string, scopeID string) (*ScopeRepresentation, error)
	GetScopes(token string, realm string, clientID string, params GetScopeParams) ([]*ScopeRepresentation, error)
	CreateScope(token string, realm string, clientID string, scope ScopeRepresentation) (*ScopeRepresentation, error)
	UpdateScope(token string, realm string, clientID string, resource ScopeRepresentation) error
	DeleteScope(token string, realm string, clientID string, scopeID string) error

	GetPolicy(token string, realm string, clientID string, policyID string) (*PolicyRepresentation, error)
	GetPolicies(token string, realm string, clientID string, params GetPolicyParams) ([]*PolicyRepresentation, error)
	CreatePolicy(token string, realm string, clientID string, policy PolicyRepresentation) (*PolicyRepresentation, error)
	UpdatePolicy(token string, realm string, clientID string, policy PolicyRepresentation) error
	DeletePolicy(token string, realm string, clientID string, policyID string) error

	GetPermission(token string, realm string, clientID string, permissionID string) (*PermissionRepresentation, error)
	GetPermissions(token string, realm string, clientID string, params GetPermissionParams) ([]*PermissionRepresentation, error)
	CreatePermission(token string, realm string, clientID string, permission PermissionRepresentation) (*PermissionRepresentation, error)
	UpdatePermission(token string, realm string, clientID string, permission PermissionRepresentation) error
	DeletePermission(token string, realm string, clientID string, permissionID string) error
}
```

## Configure gocloak to skip TLS Insecure Verification

```go
    client := gocloak.NewClient(serverURL)
    restyClient := client.RestyClient()
    restyClient.SetDebug(true)
    restyClient.SetTLSClientConfig(&tls.Config{ InsecureSkipVerify: true }
```

## developing & testing
For local testing you need to start a docker container. Simply run following commands prior to starting the tests:

```bash
docker pull quay.io/keycloak/keycloak
docker run -d \
	-e KEYCLOAK_USER=admin \
	-e KEYCLOAK_PASSWORD=secret \
	-e KEYCLOAK_IMPORT=/tmp/gocloak-realm.json \
	-v "`pwd`/testdata/gocloak-realm.json:/tmp/gocloak-realm.json" \
	-p 8080:8080 \
	--name gocloak-test \
	quay.io/keycloak/keycloak:latest -Dkeycloak.profile.feature.upload_scripts=enabled

go test
```

Or you can run the tests on you own keycloak:
```bash
export GOCLOAK_TEST_CONFIG=/path/to/gocloak/config.json
```

All resources created as a result of unit tests will be deleted, except for the test user defined in the configuration file.

To remove running docker container after completion of tests:

```bash
docker stop gocloak-test
docker rm gocloak-test
```

## License
[![FOSSA Status](https://app.fossa.io/api/projects/git%2Bgithub.com%2FNerzal%2Fgocloak.svg?type=large)](https://app.fossa.io/projects/git%2Bgithub.com%2FNerzal%2Fgocloak?ref=badge_large)
