## Bittorent Fileshare Notes for P2P - Initial Structure and Syntax

Pieces and Blocks:
- torrent divided into multiple pieces, pieces -> blocks
- Note: Piece Formula: 
	- fixed_piece_size = size_of_torrent / number_of_pieces
- Note: Block Formula:
	- number_of_blocks = (fixed_piece_size / fixed_block_size) + !!(fixed_piece_size % fixed_block_size)
- Note: Block Index Formula:
	- block_index = block_offset % fixed_block_size

Bencoding: 
- string: length prefix base 10 ( 4:spam = spam )
- integer: i prefix, e suffix ( i-3e = -3 )
- list ( array ):i prefix, e suffix ( l4:spam4:eggse = [spam, eggs] )
- Dictionary ( object ) : d prefix follow by key value pairs ( d3:cow3:moo4:spam4:eggse = {'cow': 'moo', 'spam': 'eggs'} )

MetaInfo:
- bencoded dictionary contains:
	- structure:
		-	announce - str - required
		-	announce-list - str list - optional
		-	comment - string - optional
		-	created by - optional 
		-	creation date - str - optional
		-	info - required - key - pointer - 
			- Info for Single:
				-	length - int
				-	announce-list - optional
				-	comment -optional
				-	created by - optional 
				-	creation date - optional
				-	info - required - pointer
			-  Info for Multi:
				-	files - list of dictionaries - req
					-	length - int (bytes) - req
					-	md5sum - optional 32 char str MD5 sum of file
					-	path - list of strings - relative to topmost directory, lest element is name, preceeding dictate hirearchy 
				-	name - string (topmost)
				-	piece length - int (bytes in each piece)
				-	pieces - Concatenation of all 20-byte SHA1. "The first 20 bytes of the string represent the SHA1 value used to verify piece index 0." - 

Tracker HTTP Protocol: (THP)
	- introduce peers
	-  may assume dead if request fails
- Request Keys: HTTP GET
	- info_hash - required
		- 20-byte SHA1 hash value.
		- "peer must calculate the SHA1 of the value of the "info" key in the metainfo file"
	- peer_id - required
		- 20 byte id of peer
	- port - required
		- listening number
	- Uploaded - Required
		- base 10 int of total bytes uploaded to swarm
	- Downloaded - Required
		- base 10 int of total bytes downloaded from swarm
	- left - Req
		- base 10 int total bytes peer needs
	- ip - optional
		- IPv4 format, hexadecimal IPv6 format, or a DNS name
	- NumWant - optional
		- num of peers wanted
	- event - optional
		- if not given, taken as regular periodic 
			- else if - must hafe: 
				- started - first http get request
				- stopped - shutdown gracefully
				- completed - sent when done , NOT if started complete
- Response: 
	- 'failure_reason': string : optional
		- human legible
	- 'interval': requried
		- number of times peer should wait for 
	- 'complete': optional
		- int of seeders 
	- 'incomplete' : optional
		- int of number peers downloading
	-  'peers' : required
		- "bencoded list of dictionaries containing a list of peers that must be contacted in order to download"

- Peer Wire Porotocol (PWP) :
	- tit-for-tat schema when dealing with remote peers that have just joined the swarm and thus have no pieces to offer
	- peer should upload the same amount that it has downloaded.
	- pipeline to saturate active requests
	- cooperate with other algorthims 
- A handshake is a string of bytes with the following structure:
| Name Length | Protocol Name | Reserved | Info Hash | Peer ID |
	- The handshake is a required message and must be the first message transmitted by the client. It is (49+len(pstr)) bytes long.

- **pstrlen**: string length of <pstr>, as a single raw byte
-   **pstr**: string identifier of the protocol
-   **reserved**: eight (8) reserved bytes. All current implementations use all zeroes. Each bit in these bytes can be used to change the behavior of the protocol.  _An email from Bram suggests that trailing bits should be used first, so that leading bits may be used to change the meaning of trailing bits._
-   **info_hash**: 20-byte SHA1 hash of the info key in the metainfo file. This is the same info_hash that is transmitted in tracker requests.
-   **peer_id**: 20-byte string used as a unique ID for the client. This is usually the same peer_id that is transmitted in tracker requests (but not always e.g. an anonymity option in Azureus).
- choke 
- unchoke
- intersested
- have
- bitfield
- Request
- Piece 
- Cancel
- end game
- peer selection strategy: 
	- after handshake: 
		- choked: true
		- interested: false
Sources: 
- http://jonas.nitro.dk/bittorrent/bittorrent-rfc.html
- http://www.bittorrent.org/beps/bep_0003.html
- https://wiki.theory.org/index.php/BitTorrentSpecification
	- Client encoding list
