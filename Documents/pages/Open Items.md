public:: false

- #+BEGIN_QUERY
   {:title "⚠️ OVERDUE"
    :query [:find (pull ?h [*])
              :in $ ?start ?today
              :where
              [?h :block/marker ?marker]
              [?p :page/journal? true]
              [?p :page/journal-day ?d]
              [(>= ?d ?start)]
              [(<= ?d ?today)]
              [(contains? #{"NOW" "LATER" "TODO" "DOING" "DEADLINE"} ?marker)]]
    :inputs [:56d :today]
    :collapsed? false} 
  #+END_QUERY
-