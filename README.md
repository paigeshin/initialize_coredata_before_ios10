# initialize_coredata_before_ios10

### NSPersistentStoreCoordinator

- Requires managed object context to initialize
- Allows to setup an underlying storage (SQLite is default)
- Responsible for executing queries on the store

### Get SQLite File

```swift
let dirPaths = NSSearchPathForDirectoriesInDomains(.documentDirectory, .userDomainMask, true)
print(dirPaths[0])
```

### Managed Object Context

- Create, load and manipulate managed objects
- A single application can have multiple managed object contexts
- Responsible for executing queries on the store

### Storage

- XML (not available in iOS)
- Binary
- SQLite
- In-Memory

# Before iOS 10, initialize persistent container

```swift
import Foundation
import CoreData

class CoreDataManager {
    
    lazy var managedObjectModel: NSManagedObjectModel = {
        guard let url = Bundle.main.url(forResource: "CoreDataModel", withExtension: "momd") else {
            fatalError("Failed to locate the CoreDataModel file!")
        }
        guard let model = NSManagedObjectModel(contentsOf: url) else {
            fatalError("Failed to load model")
        }
        return model
    }()
    
    lazy var coordinator: NSPersistentStoreCoordinator = {
        let coordinator = NSPersistentStoreCoordinator(managedObjectModel: self.managedObjectModel)
        let documentsDirectory = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask).first!
        let sqlitePath = documentsDirectory.appendingPathComponent("CoreData.sqlite")
        do {
            try coordinator.addPersistentStore(ofType: NSSQLiteStoreType, configurationName: nil, at: sqlitePath, options: nil)
        } catch {
            fatalError("Failed to create coordinator")
        }
        return coordinator
    }()
    
    lazy var viewContext: NSManagedObjectContext = {
        let context = NSManagedObjectContext(concurrencyType: .mainQueueConcurrencyType)
        context.persistentStoreCoordinator = self.coordinator
        return context
    }()
    
    func saveMovie(title: String) {
        let movie = Movie(context: self.viewContext)
        movie.title = title
        do {
            try self.viewContext.save()
        } catch {
            print(error.localizedDescription)
        }
    }
    
}
```
