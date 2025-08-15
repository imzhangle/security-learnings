when HTTP_REQUEST {
    # Define the allowed IPs
    set allowed_ips { 192.168.1.100 203.0.113.45 }

    # Normalize for case-insensitive path match
    set path [string tolower [HTTP::path]]
    set src_ip [IP::client_addr]

    # Check if request path starts with /secure/
    if { $path starts_with "/secure/" } {
        # Path matches, now check IP
        if { $src_ip in $allowed_ips } {
            return  ;# Allow request
        } else {
            reject  ;# Block request from unapproved IP
        }
    } else {
        reject  ;# Block all other paths
    }
}
