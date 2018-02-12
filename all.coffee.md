    module.exports = ({normalize_account,emit}) ->
      (doc) ->

        {disabled} = doc

        send = (k,v) ->

Returns all records, including disabled ones.
This is appropriate when listing all numbers, endpoints, etc. in a provisioning application.

          emit [null,k...], v

Return only enabled (active) local numbers.
This is appropriate when building lists of applicable, active options.

          emit [true,k...], v if not disabled

          return

        {type} = doc
        return unless type?

        the_value = doc[type]
        return unless the_value?

Compare with windy-moon's `validate_type`.

        if type is 'number'
          type = if '@' in the_value then 'local-number' else 'global-number'

        value = {}
        value[type] = the_value
        value.value = the_value
        value.disabled = disabled ? false

        {account} = doc
        account = normalize_account account if account? and normalize_account?

        switch type
          when 'device'
            {device} = doc.device

            doc.lines?.forEach ({endpoint}) ->
              return unless endpoint?
              send [{endpoint},type], value
              e = endpoint.match /^([^@]+)@(.+)$/
              if e
                [name,domain] = [e[1],e[2]]
                val =
                  value: the_value
                  device: the_value
                  endpoint: endpoint
                  endpoint_name: name
                  domain: domain
                send [{domain},type], val
                send [{endpoint},type], val

          when 'number_domain'
            yes

          when 'endpoint'

Not all endpoints have domains; some only have an IP address.

            e = the_value.match /^([^@]+)@(.+)$/
            if e
              [endpoint_name,domain] = [e[1],e[2]]
              send [{domain},type], value

            {number_domain} = doc
            if number_domain?
              send [{number_domain},type], value

            {location} = doc
            if location?
              send [{location},type], value

          when 'local-number'

            l = the_value.match /^([^@]+)@(.+)$/
            return unless l
            [number_name,number_domain] = [l[1],l[2]]

            send [{number_domain},type], value
            send [{number_domain},{number_name},type], value

            {user_database} = doc
            if user_database?
              send [{user_database},type], value

            doc.groups?.forEach (group) ->
              send [{number_domain},{group},0], value

            doc.allowed_groups?.forEach (group) ->
              send [{number_domain},{group},1], value

            doc.queues?.forEach (queue) ->
              send [{number_domain},{queue}], value

            doc.skills?.forEach (skill) ->
              send [{number_domain},{skill}], value

            if endpoint?
              send [{endpoint},type], value

              e = endpoint.match /^([^@]+)@(.+)$/
              if e
                [endpoint_name,domain] = [e[1],e[2]]
                send [{domain},{endpoint_name},type], value

View for (admin) routing of global numbers.
The view lists all global numbers for a given account.
The view lists the global number(s) routing to a given local-number.

          when 'global-number'

            {local_number} = doc
            if local_number?
              send [{local_number},type], value
              if l = local_number.match /^([^@]+)@(.+)$/
                [number_name,number_domain] = [l[1],l[2]]
                send [{number_domain},{number_name},type], value

We currently do not index types not listed above.

          else
            return

        send [{account},type], value

        {sub_account} = doc
        if sub_account?
          send [{account},{sub_account},type], value

        return
