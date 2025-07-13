# LucaERP 10_a ForEach
#LucaP
I'm building an ERP system from scratch to explain all the issues involved in making an ERP. Unfortunately, I'm late again on an installment of Luca ERP due to a bug in the code I was supposed to write. The bug is due to a Swift UI feature: my improper use of ForEach. As an aside to my usual work, I wanted to write about it in case anyone else gets confused by it. For those working in SwiftUI,  I want to explain the problem of **ForEach** with binding here and build a foundation we'll build our template on next time. 

This is the place to find the download for the works we're doing here. Be aware I did this is iOS 26 beta 3, so if you are in lower version, change the version number accordingly. 

Let's start with this table of data, which is a list of people involved with the Huli Pizza Company restaurants. 

**--Insert Image Here--**

I can convert this to a table in Swift. 

```
enum Role:String{
    case founder = "Founder"
    case vendor = "Vendor"
    case employee = "Employee"
    case customer = "Customer"
    case ohana = "Ohana"
    case regulator = "regulator"
}

struct aRow: Identifiable{
    var name: String
    var id: Int
    var role: Role = .customer
}

@Observable class AModel{
    var table:[aRow] = []
    init(){
        table = [
            aRow(name:"Nova",id:0,role:.founder),
            aRow(name:"David",id:1,role:.founder),
            aRow(name:"Keiko",id:2,role:.founder),
            aRow(name:"George",id:3,role:.founder),
            aRow(name:"Craig",id:4,role:.vendor),
            aRow(name:"Carmen",id:5,role:.ohana),
            aRow(name:"Steve",id:6, role:.regulator),
            aRow(name:"Auntie Maise",id:6, role:.ohana),
            aRow(name:"Kai",id:7),
            aRow(name:"Jesse",id:8,role:.employee),
            aRow(name:"Sara",id:9,role:.employee),
            aRow(name:"Tia",id:10,role:.employee),
            aRow(name:"Ralph",id:11,role:.employee),
            aRow(name:"Jorge",id:12,role:.employee),
            aRow(name:"Mark",id:13,role:.employee)
            
        ]
    }
}
```

In my view, I can declare this table in two ways: 

```
@State var aModel = AModel()
@State var table = AModel().table
```

The first gives me access to any other properties and methods available on AModel, while the second is the table alone. 


For the simplest use of **ForEach**, with a little formatting, we get:  

```
Text("Basic ForEach, all values").font(.title)
ScrollView{
    ForEach(table){row in
        HStack{
            Text(row.name).frame(width:150)
            Text(row.role.rawValue).frame(width:150)
            Spacer()
        }
    }
}
.padding(.bottom,10)
```

This code gives us a list of people and their roles in the company.  

**--insert image here--**

 If we wanted to edit the name, we'd need a text field. 

I'll make another copy of this list under the first using a **TextField** instead of **Text**. 


```
Text("Binding ForEach").font(.title)
ScrollView{
    ForEach(table){row in
        TextField("Name", text: row.name).frame(width:150)
        Text(row.role.rawValue).frame(width:150)
    }
}
.padding(.bottom,10)
```

However, that gets me an error: 

**`Cannot convert value of type 'String' to expected argument type 'Binding<String>'`**

The error is due to the **text:** parameter of **TextField** being Binding. A @Binding variable can send data down the hierarchy into subviews and, if changed there, reflect that value up the view hierarchy. In most cases, you signify something is binding with a **$** prefix to a **@State** or another **@Binding** variable. 

```
TextField("Name",text:$row.name).frame(width:150)
```

Making this change gives me a new error message
**`Cannot find '$row' in scope`**

The **row** in My ForEach is neither a binding nor a state variable, so the binding version doesn't exist. I have to declare this as binding. SwiftUI lets me declare this in one place: the object I'm iterating over that can be binding, in this case,  **table**. 

Depending on the Xcode version, you get different error messages. In playgrounds on iOS 18, I got 

**`Generic parameter 'V' could not be inferred`**

and in Xcode26 Beta 3

**`Cannot assign to property: 'rawValue' is immutable`**
**`Initializer 'init(_:)' requires that 'Binding<String>' conform to 'StringProtocol'`**

Both errors mean the same thing, though Xcode26 explains it better on the first line. We've satisfied the binding for the **text:** parameter, but now that **row** is binding, something that isn't binding like the role's **rawValue**, which is a constant, doesn't like it. 

There are two ways of handling this. One is to use a wrapped value on these non-binding values. 

```
Text(row.wrappedValue.role.rawValue).frame(width:150)
```

The other is to indicate that **row** is binding with **$row**

```
ForEach($table){$row in
    HStack{
        TextField("Name",text:$row.name).frame(width:150)
        Text(row.role.rawValue).frame(width:150)
    }
}
```

This second code snippet is more consistent with SwiftUI code and, thus, preferred. 

While I can't replicate it here, it also confuses the compiler, giving the unable to evaluate type message. 



With the code we've written, we can edit the list. I can change David to Chef David and Carmen to her *nom de gurre* for as a roller derby blocker for example. 

**--Insert Image Here--**

I'm going to add two more features as part of the foundation of the hybrid template. 

I'll add another variable to indicate selected rows. 

``` 
@State var selected: Int! = nil
```

I'll make every row a button, which, when tapped, selects the row, highlighting it. 

```
Button{
    selected = row.id
} label:{
    HStack{
        TextField("Name", text: row.name).frame(width:150)
        Text(row.wrappedValue.role.rawValue).frame(width:150)
        Spacer()
    }
    .background(.yellow.opacity(selected == row.id ? 1.0 : 0.01))
}
```

This highlighting works well, making it easier to see which row we're working with. 

**--Insert Image Here--**

We have two instances of the table, **aModel.table** and **table**. If I use **aModel.table** in the lower half and **table** in the upper half, I will see no updating in the upper half, since it is a *different table*. Make sure those match up. 

Let's say I want to show only founders. I could use a filter to do that. The **filter** method takes a predicate to show only true cases for the predicate. I'll discuss predicates in more detail in the next newsletter, but they are closures that return a Boolean value.
I'll change the top **ForEach** to use **aModel** and filter for founders. 

```
ForEach(aModel.table.filter{$0.role == .founder}){row in
```

That code gets me founders as a read-only list. I'll do the same on the bottom one,

```
ForEach($aModel.table.filter(predicate)){row in
```

 All hell breaks loose on error messages. 

**`Cannot call value of non-function type 'Binding<@Sendable ((aRow) throws -> Bool) throws -> [aRow]>'`**
**`Cannot infer contextual base in reference to member 'founder'`**
**`Dynamic key path member lookup cannot refer to instance method 'filter'`**

The problem here is the same as before. I used **$0** to designate the row in the filter, but **row** is not a binding variable. I have to write this differently to make sure I'm comparing to a binding variable. 

```
ForEach($aModel.table.filter{$row in row.role == .founder}){$row in
```

Which filters the binding rows. The binding rows with a filter have one other purpose - as a child table. For instance, consider an app that searches for people based on their role at Huli Pizza Company. Our role table looks like this. 

**--Insert Image Here--**

It will use our **aModel** data as a child, showing only the people with that role. 

I'll add this model to the app:

```
struct RoleDescriptor: Identifiable{
    var id: Role
    var description: String
}

class Roles{
    var roles:[RoleDescriptor] = [
        RoleDescriptor(id: .founder, description: "The original investors in HPC"),
        RoleDescriptor(id: .vendor, description: "People who we pay for goods and services(A/P)"),
        RoleDescriptor(id: .employee, description: "People we pay to work in our facilities"),
        RoleDescriptor(id: .ohana, description: "Friends and family"),
        RoleDescriptor(id: .regulator, description: "Local, State and Federal Government people. "),
        RoleDescriptor(id: .customer, description: "People who love our pizza so much, they give us money for it"),
        
    ]
}        
```

We have role descriptors, based on the Enum **Role**. I'll select a role by moving through these individually, showing the people in each of these role categories. The role becomes the parent, and the model becomes a child view. 

I'll add another state variable for the roles and an array index. Usually I'd use a picker for this, but for illustration purposes I want to show one at a time, and so I'll work with the array indices since the **id** is non-numeric. 


```
@State var roles = Roles().roles
@State var index:Int = 0
```

At the top of the **VStack**, I'll add this to display my new parent data:
 
 ```
Text(roles[index].id.rawValue)
    .font(.largeTitle).bold()
Text(roles[index].description)
Divider()
```

The child views use **role[index].id** in their predicates, filtering to the current role only. 

```
ForEach(aModel.table.filter{$0.role == roles[index].id}){row in
...
ForEach($aModel.table.filter{$row in row.role == roles[index].id}){$row in
```

 And finally, a next button at the bottom. 
 
 ```
Button("Next"){
	index = (index + 1) % roles.count
}
.font(.title)
.buttonStyle(.borderedProminent)
 ```
 
We have the basics of a parent with the child tables changing with it. This code is the basis for displaying many types of forms, like invoices, bills of materials, and sales orders. A template for this will be the core of LucaERP's hybrid template. 

While preparing this newsletter, I encountered an issue that prevented me from providing the hybrid template as intended. However, I got to explore with you SwiftUI and how **ForEach** works with binding variables, and why you need to be careful to track all of them in a **ForEach** loop. 
 
In the next LucaP newsletter, we'll look at the rest of the template and put it together, based on this foundation. We have a lot to discuss when we start talking about dependencies of parent-child tables adopting Crudable, flexibility in implementation, and getting the best user experience in a tablet environment. 
