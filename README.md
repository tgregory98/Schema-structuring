# **Schema-dreamer**
# Release 0.1.0. Data extraction and transformation.
**Builds, cleans and enriches "schema" diagrams, connecting articles from across Wikipedia via the RDF DBpedia dataset.**

A Python program which queries DBpedia's RDF dataset, a community maintained dataset based off Wikipedia, via the SPARQL endpoint. Then it uses this data to intelligently build "schema" representing the connections between articles from across Wikipedia. These schema are built with Cypher on the Neo4j graph database platform.

The main aims of this program were to successfully extract the data, transform it, and then to rapidly find clear schema representations. Loading into a cloud data warehouse may follow soon (to complete the ETL process). This program will be the first stage of a broader project inspired by the roots of [constructivism](https://en.wikipedia.org/wiki/Constructivism_(philosophy_of_education)) (or more specifically [schemas](https://en.wikipedia.org/wiki/Schema_(psychology)), which are *not* to be confused with database schemas).

I took four alternative approaches, all of which are listed below in decreasing order of effectiveness. Most queries (in both SPARQL and Cypher) are dynamically generated based off the desired depth and the chosen root node. Where possible, I have also written built-in filter options. It all runs from one file using class instances, and it can go from a blank slate to a fully populated final schema in a matter of seconds.

## File/ folder structure
- **demo_schemas**: Contains the image results of some of the possible approaches.
- **modules**: Contains the scripts which do most of the heavylifting.
    - builders.py: (extract) The main script responsible for querying and building the initial graphs.
    - cleaners.py: (transform) This script removes unwanted information from the graph.
    - enrichers.py: (transform) This script makes the data in the graph more readable.
    - tr_funcs.py: This script provides database Transaction utilities for use across the project.
- TASK_LIST.md: A task list for personal use.
- run.py: The script that we 'run', and acts as a dashboard for arranging the various components of the graph we wish to build.
- style.grass: A loose file which may be uploaded to the Neo4j browser for personalised styling.

## Showcase
Screenshots of some of the outputs generated by the three alternative approaches, ran on three different Wikipedia articles: Netflix, Television and Smart_TV.
#### 1. ParentSchemaBuilder + DisjointParentSchemaCleaner (depth=2, root_nodes=3) (8.3 seconds)
![image](https://github.com/tgregory98/Schema-structure/blob/master/demo_schemas/ParentSchemaBuilder%20%2B%20DisjointParentSchemaCleaner%20(depth%3D2%2C%20root_nodes%3D3)%20(8.3%20seconds).png)

This algorithm performs the best in terms of clarity of representation, speed and scalability. It finds parent categories until the root node trees intersect and we get a connection (or rather, these branches get temporarily tagged and so they don't get removed by the cleaner class). Going forward, this will be the algorithm of choice.

#### 2. ParentSchemaBuilder + LeafSchemaCleaner (depth=2, root_nodes=3) (5.4 seconds)
![image](https://github.com/tgregory98/Schema-structure/blob/master/demo_schemas/ParentSchemaBuilder%20%2B%20LeafSchemaCleaner%20(depth%3D2%2C%20root_nodes%3D3)%20(5.4%20seconds).png)

If you replace the DisjointParentSchemaCleaner from above with LeafSchemaCleaner then you can increase the speed of the first algorithm. This one finds parent categories and then iteratively removes leafs for a fixed number of passes. It is only the trees attached to a singular root that are removed by LeafSchemaCleaner. The result is similar to the above, except it preserves clusters of nodes around singular roots.

#### 3. PairwiseSchemaBuilder (depth=2,3,4, root_nodes=2) (5.0 seconds)
![image](https://github.com/tgregory98/Schema-structure/blob/master/demo_schemas/PairwiseSchemaBuilder%20(depth%3D2%2C3%2C4%2C%20root_nodes%3D2)%20(5.0%20seconds).png)

This one isn't quite as useful as the previous two in my opinion because the representations aren't as meaningful nor as scalable. Sometimes the complexity explodes because it looks at both children and parents.

#### 4. PopulateSchemaBuilder + DisjointParentSchemaCleaner (depth=1, root_nodes=3) (8.5 seconds)
![image](https://github.com/tgregory98/Schema-structure/blob/master/demo_schemas/PopulateSchemaBuilder%20%2B%20DisjointParentSchemaCleaner%20(depth%3D1%2C%20root_nodes%3D3)%20(8.5%20seconds).png)

Finally, we have the least effective approach. This is essentially what happens if you run the first approach, except you consider not just parents but also children. The complexity explodes very fast so I had to limit the depth of this search to 1. The only reason I have kept this around is because it is sometimes useful for data exploration.

