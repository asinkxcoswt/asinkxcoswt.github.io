# You should make everying a pettern
# Checklist of a pattern (test, etc.)
# pattern design phase
# you have only pencil, but the requirement ask you to make a Vangoh like drawing. You don't head up to the work. You should arrange a meeting and discuss about the new tools to use.

# management team will see the problem together with development team.

# What make up a platform?
# Build up thing on top of a platform is fast.
# Build the platform itself is slow.

##gihook


#########################

# Misleading from Spring Framework software design tutorial out there

# usual layered achitecture, and its problems
# 


# What they didn't tell you about layered architecture.
# Layered architecutre is the most basic and should be a default architecture when you don't know other ways
# nevertheless you should learn about other architecture

# layered architecutre is for separate work. To decide the layer to have, you should analyze if the work is worth to be separate to a layer, a layer has cost.

# Example, in KKP project, we have a node is system architecutre as a REST API, in this node, we use typical layered architecture. It's worthless. 



# what can be wrong for novice using layered architecture.
    - useless interface
    - no clear seperation of concern
    - no test on layer's interface
    - the big interface
    - the one impl class


# wasteful service layer
    - http://johannesbrodwall.com/2014/07/10/the-madness-of-layered-architecture/
    - We confuse with data access layer, in the old time, accessing database has a lot of work and the application layer should not concern with the work. But today, JPA repostiory has no such work at all.
    - JPA Repository is just a kind of service


# possible better why: spring data rest, http invoker, RMI