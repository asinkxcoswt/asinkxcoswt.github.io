# ความเข้าใจผิดเกี่ยวกับ Dependency Injection และ Dependency Inversion Principle

# Dependency Injection helps about inversion of control
    it means instead of instantiating an objection in side a class, we leave it to the framework to manage lifecycle of the object
    it enable the framework to enhance the bean in many way

# Just we using DI, doesn't mean we employ DIP
    Ownership of the interface.

# We don't always have to inject dependency through interfaces.
    Confusion with DIP
    Controler, port & adaptor
    Some components do not have business dependency
    Programming to interface useful for testing, but we don't test everything! The adaptor class is an example where I think it has very least value to test.
        adaptor itself is a mean to decouple 2 elements, it is a kind of compoonent that is design to be changed when one side of the depdency change. No need to create interface for it!
