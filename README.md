# Support template identifiers

## Concept

It would be beneficial to pass a string literal as a template argument. This string literal could be used in place of regular identifier, i.e member field name, member function name, function argument name. To unleash the full power of the concept it should be possible to bind a string literal alias like in case of regular types (e.g. 'using custom_alias = "string_literal";'). Together with variadic templates 'new aliases' could be used to define member fields, member functions and in context of member initialization list. This would lead to new rules of extension parameters pack (see examples).

## Rationale

- Less macros,
- More verbose code,
- More code that can be reused in different context, without being so misty like 'first', 'second'.
- No need to workaround with multiple inheritence - access member fields directly.
- Easy to write non-intrusive polymorphic wrappers around included/imported types.

## Intro

The most fundamental concept:
```
template<typename T, identifier A>
struct TypeAlias {
    using type = T;
    using alias = A;
};
```

## Examples

First use:
```
template<typename First, typename Second>
struct NamedPair {
    First::type First::alias;
    Second::type Second::alias;
};
```

Basic example:
```
template<typename KeyType, typename ValueType>
using KeyValue = NamedPair<TypeAlias<KeyType, "key">, TypeAlias<ValueType, "value">>;
```

Named tuple (with member field expansion):
```
template<typename... TypeAlias>
struct NamedTuple {
    TypeAlias::type TypeAlias::alias;...
};
```

More complex example (with member initializer list expansion #1 and member function expansions #2):
```
template<typename T, identifier A, identifier G, identifier S, identifier V>
struct TypeAliasGetterSetter {
    using type = T;
    using alias = A;
    using getter = G;
    using setter = S;
    using var = V;
};

template<typename... TAGS>
struct StrictTuple {
    StrictTuple(TAGS::type TAGS::var...) : TAGS::alias(TAGS::var)... {}                       // #1
    void TAGS::setter(TAGS::type& TAGS::var) { this->identifier TAGS::alias = TAGS::var; }... // #2
    TAGS::type TAGS::getter() { return this->identifier TAGS::alias; }...                     // #2
private:
    TAGS::type TAGS::alias;...
}
```

## Natural extension 
If simple identifier concatenation would be allowed, the previos code could be simplified:
```
template<typename T, identifier A>
struct TypeAliasGetterSetter {
    using type = T;
    using alias = "m_" + A;
    using getter = "get_" + A;
    using setter = "set_" + A;
    using var = A;
};
```

Having these declaration we could use the template:
```
template<typename T, identifier A>
using TAGS = TypeAliasGetterSetter;

using MyStrictTuple = StrictTuple<TAGS<int, "x">, TAGS<int, "y">, TAGS<std::string, "label">>;

MyStrictTuple mst(1,2,"hello");
assert(mst.get_x() == 1)
mst.set_y(7);
assert(mst.get_y() == 7);
assert(mst.get_label()[0] == 'h');
```

## Last but not least

```
template <typename T>
void delegated1(T&);
...

template <typename T>
void delegated2(T&, int);
...

template<identifier id_delegate_1, identifier id_delegate_2>
struct Delegate {
    template<typename T>
    Delegate(T t) : store(std::make_unique<model_t<T>>(t)) {}
    
    void id_delegate_1() {store->id_delegate_1();};
    void id_delegate_2(int x) {store->id_delegate_2(x);};
    
private:
struct concept_t {
    virtual ~concept_t() = default;
    virtual void id_delegate_1() = 0;
    virtual void id_delegate_2(int) = 0;
}

template<T>
struct model_t : public concept_t {
    model_t(T t) : t(t) {}
    void id_delegate_1() { ::id_delegate_1<T>(t); };
    void id_delegate_2(int x) { ::id_delegate_2<T>(t,x); };
    T t;
}

std::unique_ptr<concept_t> store;
}

std::vector<Delegate<"delegated1", "delegated2">> collection;
...
for(auto& item : collection) {
  item.delegated1();
  item.delegated2(7);
}
```

Here the class Delegate can be written once. The signatures of delegate1 and delegate2 can be encapsulated in dedicated class, as well as their specializations. It is just an example, but it could be written even more concise and verbose, allowing variadic number of methods, with any signatures.
