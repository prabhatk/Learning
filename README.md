import Foundation

class WidgetDownloadManager {
    static let shared = WidgetDownloadManager()
    
    private init() {}
    
    // MARK: - Base Directories for Each Widget Type
    private func baseDirectory(for widgetType: String) -> URL? {
        let fileManager = FileManager.default
        guard let containerURL = fileManager.containerURL(forSecurityApplicationGroupIdentifier: "group.com.yourapp.shared") else {
            return nil
        }
        return containerURL.appendingPathComponent(widgetType)
    }
    
    // MARK: - Data Save Locations for Each Size
    private func filePath(for widgetType: String, size: WidgetSize) -> URL? {
        guard let baseDir = baseDirectory(for: widgetType) else { return nil }
        return baseDir.appendingPathComponent(size.rawValue)
    }
    
    // MARK: - Download Handlers
    func downloadNewsData(size: WidgetSize, completion: @escaping (Result<Data, Error>) -> Void) {
        let url = URL(string: "https://example.com/news?size=\(size.rawValue)")!
        downloadData(from: url, widgetType: "news", size: size, completion: completion)
    }
    
    func downloadScoresData(size: WidgetSize, completion: @escaping (Result<Data, Error>) -> Void) {
        let url = URL(string: "https://example.com/scores?size=\(size.rawValue)")!
        downloadData(from: url, widgetType: "scores", size: size, completion: completion)
    }
    
    func downloadTeamData(size: WidgetSize, completion: @escaping (Result<Data, Error>) -> Void) {
        let url = URL(string: "https://example.com/team?size=\(size.rawValue)")!
        downloadData(from: url, widgetType: "team", size: size, completion: completion)
    }
    
    // MARK: - Core Download Method
    private func downloadData(from url: URL, widgetType: String, size: WidgetSize, completion: @escaping (Result<Data, Error>) -> Void) {
        let task = URLSession.shared.dataTask(with: url) { data, response, error in
            if let error = error {
                completion(.failure(error))
                return
            }
            
            guard let data = data else {
                completion(.failure(NSError(domain: "", code: 0, userInfo: [NSLocalizedDescriptionKey: "No data found"])))
                return
            }
            
            // Save data to the correct place for the widget type and size
            if let fileURL = self.filePath(for: widgetType, size: size) {
                do {
                    try data.write(to: fileURL)
                    print("Data saved to: \(fileURL)")
                } catch {
                    completion(.failure(error))
                }
            }
            
            // Cache the data
            CacheManager.shared.setObject(data as NSData, forKey: "\(widgetType)-\(size.rawValue)" as NSString)
            
            completion(.success(data))
        }
        task.resume()
    }
    
    // MARK: - Read Data
    func readData(for widgetType: String, size: WidgetSize) -> Data? {
        if let filePath = filePath(for: widgetType, size: size), let data = try? Data(contentsOf: filePath) {
            return data
        }
        return nil
    }
}

// MARK: - WidgetSize Enum
enum WidgetSize: String {
    case small = "small"
    case medium = "medium"
    case large = "large"
}



import Foundation

class CacheManager {
    static let shared = CacheManager()
    
    private let cache = NSCache<NSString, NSData>()
    
    private init() {}
    
    // Save data to cache
    func setObject(_ data: NSData, forKey key: NSString) {
        cache.setObject(data, forKey: key)
    }
    
    // Retrieve data from cache
    func object(forKey key: NSString) -> NSData? {
        return cache.object(forKey: key)
    }
    
    // Clear cache, with an option to retain specific keys
    func clearCache(except keysToRetain: [NSString]) {
        let allKeys = cache.name.enumerate().map { $0.key }
        
        for key in allKeys {
            if !keysToRetain.contains(key as NSString) {
                cache.removeObject(forKey: key as NSString)
            }
        }
    }
}
\//usage


WidgetDownloadManager.shared.downloadNewsData(size: .small) { result in
    switch result {
    case .success(let data):
        print("News data downloaded for small widget: \(data)")
    case .failure(let error):
        print("Failed to download news data: \(error)")
    }
}

WidgetDownloadManager.shared.downloadScoresData(size: .medium) { result in
    switch result {
    case .success(let data):
        print("Scores data downloaded for medium widget: \(data)")
    case .failure(let error):
        print("Failed to download scores data: \(error)")
    }
}
