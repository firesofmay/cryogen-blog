{:title "Manipulating vector of maps - Clojure"
 :layout :post
 :tags  ["clojure"]}

So here is a vector of maps.
Map contains name, age and school as keys. *Note* that school is an optional key.

```clojure
(def student-grades [{:age 21, :name "Frank", :school "ABC"}
                     {:age 23, :name "Shawn"}
                     {:age 11, :name "Eliot", :school "XYZ"}])
```

Now I would like to take that vector of maps and return the same name, increment the age and if school key is missing return `-` as its value else return the school itself.
So I write a function which take one map like that and de-structure its keys.

```clojure
(defn get-grades
  "This will take a map of name age school as keys and
   return a map of name, incremented age and
   if school is not given then '-'
   else the name of the school itself."
  [{:keys [name age school]}]
  {:name name
   :age (inc age)
   :school (or school "-")})
```

And now all I have to do is map that function onto that whole vector of maps and we get

```clojure
(map get-grades
     student-grades)
({:name "Frank", :age 22, :school "ABC"}
 {:name "Shawn", :age 24, :school "-"}
 {:name "Eliot", :age 12, :school "XYZ"})
```

Notice age has been incremented, and if school is empty it is returning a `-`