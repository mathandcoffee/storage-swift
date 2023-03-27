# `storage-swift`

Swift Client library to interact with Supabase Storage.

## `Prelude: RLS`

A lot of bug reports get filed towards all the Supabase Storage libraries because
RLS is easy to get wrong for the un-initiated.

Before you can use any client service, you should follow the RLS guides on 
Supabase's website. You will need to create policies on each bucket individually
or use global ones on the `storage/object` and `storage/bucket` sections.

For image/video uploads from users, there is a key example on how to setup basic
RLS for all CRUD operations [here]`https://supabase.com/docs/guides/storage/access-control`
Pay particular attention to the difference between `USING` and `WITH CHECK` keyword
for POSTGRES SQL.

Once you have RLS setup for ALL CRUD operations you should be able to use the
bucket with the clients. There are some issues with uploading or downloading with
buckets that change their permission levels from public to private and vice versa,
so use caution when playing around with those settings if you're experimenting. If
you experience a bunch of 404's with uploads and downloads, try deleting the
bucket and re-initializing it.

## `Instantiating the client`

If you're using the full `supabase-swift` experience, you can create a reference
to the storage client by accessing the following:

```Swift
import Supabase
import SupabaseStorage

lazy var supabaseClient: SupabaseClient = {
    return SupabaseClient(supabaseURL: supabaseUrl, supabaseKey: apiKey)
}()

lazy var supabaseStorageClient: SupabaseStorageClient = {
    return supabaseClient.storage
}()
```

If you're experiencing errors using this instance, or would just like to construct 
your own, you may do so via:

```Swift
func storageClient(bucketName: String = "photos") async -> StorageFileApi? {
    guard let jwt = try? await supabaseClient.auth.session.accessToken else { return nil}
    return SupabaseStorageClient(
        url: "\(supabaseUrl)/storage/v1",
        headers: [
            "Authorization": "Bearer \(jwt)",
            "apikey": apiKey,
        ]
    ).from(id: bucketName)
}
```

## `Uploading and Downloading`

With your client of choice, you can download via the following given some RLS and 
naming conventions illustrated above.

Example of converting a `.png` image download to a `UIImage` for a `UIImageView`:

```Swift
if let data = try? await SupabaseProvider.shared.storageClient()?.download(
    path: profilePhotoUrl
) {
    imageView.image = UIImage(data: data)
}
```

Example of uploading an image via a `UIImageView's` current `UIImage` and saving 
it to a user's bucket folder:

```Swift

func loggedInUserId() async -> String? {
    return try? await supabaseClient.auth.session.user.id.uuidString
}

guard let image = imageView.image?.pngData() else { return }

// Note that Swift has UUID's all capitalized, but Supabase will always lowercase
// them.

guard let userId = loggedInUserId().lowercased() else { return }

let fileId = UUID().uuidString
let filename = "\(fileId).png"
let storageClient = await SupabaseProvider.shared.storageClient()
guard let uploadResponseData = try? await storageClient?.upload(
        path: "\(userId)/\(filename)", 
        file: File(
            name: filename, 
            data: image, 
            fileName: filename, 
            contentType: "image/png"), 
            fileOptions: FileOptions(cacheControl: "3600")
        )
    ) else { return }
```

## `URL Creation`


## Sponsors

We are building the features of Firebase using enterprise-grade, open source products. We support existing communities wherever possible, and if the products don’t exist we build them and open source them ourselves. Thanks to these sponsors who are making the OSS ecosystem better for everyone.

[![New Sponsor](https://user-images.githubusercontent.com/10214025/90518111-e74bbb00-e198-11ea-8f88-c9e3c1aa4b5b.png)](https://github.com/sponsors/supabase)
