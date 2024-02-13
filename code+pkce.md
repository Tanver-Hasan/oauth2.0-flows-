@startuml
!theme plain
participant User
participant Client
participant AuthorizationServer
participant ResourceServer
autonumber
User -> Client: Initiate Authorization Request 
Client -> Client: Generate cryptographically random code_verifier 
Client -> Client: Generate code_challenge  from code_verifier
Client -> AuthorizationServer: Redirect to Authorization \n Server(/authorize)  with  code_challenge
AuthorizationServer -> User: Authenticate User
User -> AuthorizationServer: Authorize Client
AuthorizationServer -> Client: Authorization Code 
Client -> AuthorizationServer: Token Request  (/oauth/token) with \n Authorization Code  and  code_verifier
AuthorizationServer -> AuthorizationServer: Validate  code_verifier \n and  code_challange
AuthorizationServer -> Client: idToken and Access Token
Client -> ResourceServer: Access Token
ResourceServer -> Client: Protected Resource
@enduml
