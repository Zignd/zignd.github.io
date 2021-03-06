---
layout: post
title:  "Using Clojure's prismatic/schema to easily convert data from one schema to another"
date:   2020-01-11 00:00:00 -0300
categories: clojure schema
---
Sometimes you might end up finding yourself in a situation in which you have data in one format because it came from a database query and you might want to convert it to another format, which is used throughout the system. Even though the data, in essence, is the same, there might be subtle details which require you to rename a few keys or convert a few values in order to be able to use it.

This kind of code is usually dull to write because you will find yourself repeating the same patterns quite often. There's a namespace in `prismatic/schema` which will help you do just that, and it got me excited the first time I saw its capabilities. What you're going to see below is an example of its usage.

Jekyll also offers powerful support for code snippets:

{% highlight clojure %}
(ns example
  (:require [schema.core :as s]
            [schema.coerce :as coerce])
  (:import [org.joda.time DateTime]
           [java.sql Date]))

(s/defschema DBUser
  {:id s/Int
   :username s/Str
   :password s/Str
   :type_id s/Int
   :created_at java.sql.Date})

(s/defschema User
  {:id s/Int
   :username s/Str
   :password s/Str
   :type-id s/Int
   :created-at org.joda.time.DateTime})

(defn DBUser->User
  [{:keys [id username password type_id created_at] :as db-user}]
  {:id id
   :username username
   :password password
   :type-id type_id
   :created-at created_at})

(def DBUser->User-coercer
  ;; the map defines converters on the right side for the types on the on the left side
  (coerce/coercer User {User DBUser->User
                        org.joda.time.DateTime #(coerce/from-sql-date %)}))
;; during the conversion below there will a value of type java.sql.Date in :created-at
;; above we define a converter for org.joda.time.DateTime which will be used
;; to values that should be of this type

(DBUser->User-coercer {:id 1
                       :username "mrparrot"
                       :password "a987a9s8df59"
                       :type_id 3
                       :created_at (new java.sql.Date 0)})
{% endhighlight %}
