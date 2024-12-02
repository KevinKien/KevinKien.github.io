---
layout: post
title: SQL Injection in Go
date: 2024-12-02
categories: ["Golang", "1-Day-Analysis"]
---

# Summary

SQL Injection is critical vulnerability allow attacker dump all data in database and steal all sensitive infomation of admin and customer. Additional, if database allow write permission, hacker can write malicious file and RCE system. 

In go lang, SQL injection also no exception, if developer not validate input or use query to database with string concatenate then sql injection will appear. Today, i will understand about sql injection in go lang, Where vulnerability will appear? And how to fix this?

# Struct of API App in golang

In the App golang often use struct such as: 
- Route: URL API and call to function controller
- Controller: This is function handle input from url API and data, status to response.
- Services: This is function handle logic of application, validate data.
- Repository: handle all database interacts including query, insert, delete, update, ...

This is struct be divided in to small file, have some developer will combine Controller vs Services to one file to easy manage. 

So to find vulnerability sql injection, I will focus to file Repository. At here, i alway check all query statement, if have query statement use string concatenate i will continue check at Services file. At services file if input not validate, maybe is vulnerability. To determine vulnerability is true or false, I will use sqlmap tool to check. 

# Analysis CVE-2024-45794

In the CVE-2024-45794, at file repository developer use string concatenate when query to database: 

```
func (impl UserAuthRepositoryImpl) GetRoleForChartGroupEntity(entity, app, act, accessType string) (RoleModel, error) {
	var model RoleModel
	var err error
	if len(app) > 0 && act == "update" {
		query := "SELECT role.* FROM roles role WHERE role.entity = ? AND role.entity_name=? AND role.action=?"
		if len(accessType) == 0 {
			query = query + " and role.access_type is NULL"
		} else {
			query += " and role.access_type='" + accessType + "'"
		}
		_, err = impl.dbConnection.Query(&model, query, entity, app, act)
	} else if app == "" {
		query := "SELECT role.* FROM roles role WHERE role.entity = ? AND role.action=?"
		if len(accessType) == 0 {
			query = query + " and role.access_type is NULL"
		} else {
			query += " and role.access_type='" + accessType + "'"
		}
		_, err = impl.dbConnection.Query(&model, query, entity, act)
	}
	if err != nil {
		impl.Logger.Errorw("error in getting role for chart group entity", "err", err, "entity", entity, "app", app, "act", act, "accessType", accessType)
	}
	return model, err
}
```

Vulnerability at `query += " and role.access_type='" + accessType + "'"`. Because, attacker can insert malicious payload as `' or 1=1-- -`.
So query statement like `query += " and role.access_type='" + ' or 1=1-- - + "'"` => `and role.access_type='' or 1=1-- -`.
And with payloads malicious attacker can get all data.

# Recommendation
To the fix sql injection in golang, when query statement don't use string concatenate. Please use prepare statement like: 
```
query += " and role.access_type = ? "
queryParams = append(queryParams, accessType)
```
